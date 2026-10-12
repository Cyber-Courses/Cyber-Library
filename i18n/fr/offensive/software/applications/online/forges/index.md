---
title: "Forges : attaquer les plateformes SaaS d'hébergement de code"
order: 3
description: "Attaquer les forges de code hébergées (GitHub, GitLab, Bitbucket) en tant que services opérés par un éditeur : reconnaissance et collecte de secrets à travers les organisations et dépôts, injection de pipeline CI/CD, prise de contrôle de runner auto-hébergé, et abus de jetons, d'OAuth apps et de webhooks qui transforment l'accès à un dépôt en exécution de code et compromission de la supply-chain."
keywords:
  - forge de code
  - GitHub
  - GitLab
  - Bitbucket
  - CI/CD
  - supply chain
---

# Forges

Une forge de code héberge le code source d'une organisation, son automatisation et les identifiants avec lesquels cette automatisation s'exécute, de sorte qu'en compromettre une est une voie directe vers la supply-chain logicielle. Ce sont des plateformes SaaS opérées par un éditeur : il n'y a pas de serveur à exploiter, vous travaillez via l'interface web, les API REST et GraphQL, et surtout le système CI/CD, qui exécute du code influencé par l'attaquant avec les secrets du dépôt et les identifiants de déploiement. La chaîne récurrente est la même pour les trois : trouver l'organisation et ses dépôts, récolter des secrets dans le code et l'historique, obtenir une exécution de code dans un pipeline, et transformer un jeton divulgué ou trop permissif en accès plus large.

## Triage

```bash
# Identify the forge and whether org/repo data is reachable unauthenticated
curl -s https://api.github.com/orgs/<org>            # GitHub: public org + repo metadata
curl -s https://gitlab.com/api/v4/projects?search=<org>   # GitLab: project search
curl -s https://api.bitbucket.org/2.0/repositories/<workspace>  # Bitbucket: workspace repos
# then decide the lever: exposed secrets (recon), a pipeline you can influence (CI/CD), or a token you hold
```

Choisissez le sous-sujet selon l'accès dont vous disposez : l'exposition publique et les secrets divulgués acheminent vers la **Reconnaissance** de chaque forge ; un dépôt dont vous pouvez influencer le workflow achemine vers les pages d'**injection CI/CD** et de **runner** ; un jeton ou un identifiant d'application achemine vers les pages d'**abus de jetons**.

## Sous-sujets

- **[GitHub](github/index.md)** : reconnaissance et collecte de secrets, injection de workflow Actions, prise de contrôle de runner auto-hébergé, abus de `GITHUB_TOKEN` et de PAT, et abus des OAuth Apps et des webhooks.
- **[GitLab](gitlab/index.md)** : reconnaissance, injection de pipeline `.gitlab-ci.yml`, prise de contrôle de runner, abus des jetons d'accès personnels et de job CI, et exploits admin et API connus.
- **[Bitbucket](bitbucket/index.md)** : reconnaissance, abus des Pipelines, et abus des mots de passe d'application et des jetons d'accès.

## Références

- [GitHub Actions security hardening](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [GitLab CI/CD documentation](https://docs.gitlab.com/ee/ci/)
- [PortSwigger Research : CI/CD and GitHub Actions attacks](https://portswigger.net/research)
