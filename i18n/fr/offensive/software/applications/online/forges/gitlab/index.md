---
title: "GitLab"
order: 2
description: "Attaquer GitLab en tant que forge hébergée : énumérer les groupes, projets, membres et variables CI/CD via les API REST v4 et GraphQL, injecter des jobs dans .gitlab-ci.yml, prendre le contrôle des runners partagés et à exécuteur Docker, abuser des tokens personnels, de projet, de groupe, de déploiement et de job CI, et exploiter les faiblesses publiées de prise de contrôle de compte et de RCE par upload sur les instances auto-gérées."
keywords:
  - GitLab
  - gitlab-ci.yml
  - CI job token
  - personal access token
  - runner
  - api/v4
---

# GitLab

GitLab stocke le code source d'une organisation, son automatisation et les identifiants avec lesquels cette automatisation s'exécute, c'est donc une voie directe vers la chaîne d'approvisionnement logicielle. Sur gitlab.com, tout est atteint via l'interface web, l'API REST à `https://gitlab.com/api/v4` et GraphQL à `https://gitlab.com/api/graphql` ; les instances auto-gérées exposent les mêmes API sous leur propre hôte. Le travail est organisé en **groupes** (qui s'imbriquent et possèdent l'appartenance et les variables CI/CD héritées par leurs projets) et en **projets** (des dépôts avec pipelines, registres et leurs propres variables). Le levier qui transforme l'accès en lecture en exécution, c'est la **CI/CD** : un pipeline défini par `.gitlab-ci.yml` exécute votre `script:` sur un **runner** avec les variables CI/CD du projet et du groupe dans l'environnement et un `CI_JOB_TOKEN` qui atteint l'API et le registre.

L'authentification à l'API est un en-tête, l'un de plusieurs types de token : un **personal access token** (`PRIVATE-TOKEN:`), un **token d'accès de projet/groupe**, un **deploy token**, un bearer **OAuth**, ou le **CI job token** à portée de pipeline (`JOB-TOKEN:`). N'importe lequel d'entre eux, fuité ou surdimensionné en scope, est le point de départ.

## Triage

```bash
# What a token (or anonymous access) can see
curl -s https://gitlab.com/api/v4/version -H "PRIVATE-TOKEN: $TOKEN"        # token valid + instance version
curl -s "https://gitlab.com/api/v4/groups?per_page=100" -H "PRIVATE-TOKEN: $TOKEN"
curl -s "https://gitlab.com/api/v4/projects?membership=true&per_page=100" -H "PRIVATE-TOKEN: $TOKEN"
# Anonymous public inventory (no token): public projects/groups still enumerate
curl -s "https://gitlab.com/api/v4/groups?search=<org>"
```

Un `200` avec du JSON de groupe/projet signifie que le token (ou l'accès anonyme) voit cette portée ; un `401` signifie que le token est invalide, un `403` qu'il lui manque le scope ou le rôle. Choisissez la page selon ce que vous détenez : l'exposition publique et les secrets fuités mènent à **Reconnaissance** ; une branche ou une MR dont vous pouvez exécuter le pipeline mène à **Injection de pipeline CI/CD** ; un runner que vous pouvez atteindre via un job mène à **Prise de contrôle de runner** ; un token en main mène à **Abus de token** ; une instance auto-gérée à une version connue comme vulnérable mène à **Exploits admin et API connus**.

## Pages

- **[Reconnaissance](reconnaissance.md)** : énumérer les groupes, projets, membres, snippets et variables CI/CD accessibles via v4 et GraphQL, et moissonner les secrets dans les dépôts, l'historique et les logs de jobs.
- **[Injection de pipeline CI/CD](ci-cd-pipeline-injection.md)** : exécuter un `script:` d'attaquant dans un pipeline pour exfiltrer les variables protégées et masquées et pivoter avec le `CI_JOB_TOKEN`.
- **[Abus de token](token-abuse.md)** : réutiliser des tokens personnels, de projet, de groupe, de déploiement et de job CI fuités contre l'API, en résolvant l'identité et le scope de chaque token.
- **[Exploits admin et API connus](known-admin-and-api-exploits.md)** : faiblesses publiées des instances auto-gérées par mécanisme, y compris la prise de contrôle de compte par réinitialisation de mot de passe, la RCE par upload d'image, et les classes d'IDOR et de SSRF sur l'API et GraphQL.
- **[Prise de contrôle de runner](runner-takeover.md)** : obtenir une RCE sur des runners partagés via un job contrôlé, s'évader de l'exécuteur Docker, et enregistrer un runner malveillant avec un token fuité.

## Références

- [GitLab REST API](https://docs.gitlab.com/ee/api/rest/)
- [Variables CI/CD GitLab](https://docs.gitlab.com/ee/ci/variables/)
- [GitLab GraphQL API](https://docs.gitlab.com/ee/api/graphql/)
- [PortSwigger Research](https://portswigger.net/research)
