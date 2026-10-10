---
title: "Bitbucket"
order: 3
description: "Techniques offensives contre Bitbucket Cloud en tant que cible SaaS : reconnaissance des workspaces, projets et dépôts, collecte de secrets dans le code et l'historique, exécution via Pipelines avec des variables de dépôt et sécurisées, abus de déploiement et d'OIDC, et réutilisation de mots de passe d'application et de jetons d'accès contre l'API 2.0."
keywords:
  - Bitbucket
  - Bitbucket Pipelines
  - mot de passe d'application
  - jeton d'accès
  - bitbucket-pipelines.yml
---

# Bitbucket

Bitbucket Cloud est une forge logicielle accessible via son **API REST** (base `https://api.bitbucket.org/2.0`) et le protocole `git` sur HTTPS. Sa hiérarchie est **workspace -> projet -> dépôt** : un workspace est le compte de premier niveau qui possède les dépôts et la facturation, les projets sont une couche de regroupement optionnelle à l'intérieur, et les permissions sont accordées à l'un quelconque des trois niveaux. Presque chaque attaque est une question de **ce qu'un identifiant peut lire à travers cette hiérarchie** et de **ce qui s'exécute lorsque du code entre dans un dépôt**, car **Pipelines** exécute `bitbucket-pipelines.yml` avec les variables et les identifiants de déploiement du dépôt.

Les identifiants se présentent sous quelques formes distinctes, et celle que vous détenez change tout en aval : les **mots de passe d'application** (authentification Basic, `username:app_password`, à portée par utilisateur), les **jetons d'accès de dépôt, de projet et de workspace** (authentification Bearer, à portée d'une seule ressource), les **jetons API Atlassian** (authentification Basic avec l'e-mail du compte), et les **consommateurs OAuth**. L'enchaînement récurrent est : énumérer le workspace et ses dépôts, collecter les secrets dans les clones et l'historique, obtenir l'exécution de code dans un pipeline, puis réutiliser les jetons que cette exécution expose pour pousser, déployer, et pivoter vers des comptes cloud connectés.

Ces pages ciblent `bitbucket.org` (Cloud). Bitbucket Data Center/Server est un produit différent avec une API différente (`/rest/api/1.0/`) et sans Pipelines ; les différences sont signalées en ligne là où une technique change.

## Triage

Établissez l'identité que vous détenez et la surface du workspace avant toute chose :

```bash
# What identity does this credential map to
curl -s -u "$BB_USER:$BB_APP_PASSWORD" https://api.bitbucket.org/2.0/user | jq '{username,account_id,nickname}'

# Workspaces this credential can see, then repositories in each (public + private it can reach)
curl -s -u "$BB_USER:$BB_APP_PASSWORD" https://api.bitbucket.org/2.0/workspaces | jq -r '.values[].slug'
curl -s -u "$BB_USER:$BB_APP_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/ACME?pagelen=100" | jq -r '.values[].full_name'
```

Un identifiant qui retourne des dépôts privés, ou dont l'appel `/2.0/user` réussit tout simplement, constitue déjà un point d'ancrage. Les dépôts qui contiennent un `bitbucket-pipelines.yml` sont la surface d'attaque CI ; ceux dotés d'**environnements de déploiement** configurés ou de **variables de dépôt** sont les plus précieux, car une étape de pipeline les lit en clair.

Choisissez la page selon l'accès dont vous disposez : une exposition publique, un clone que vous pouvez lire, ou une URL `git` oriente vers **Reconnaissance** ; un dépôt dont vous pouvez déclencher le pipeline oriente vers **Abus de Pipelines** ; un mot de passe d'application ou un jeton d'accès divulgué oriente vers **Abus de jetons et de mots de passe d'application**.

## Pages

- **[Reconnaissance](reconnaissance.md)** : énumérer les workspaces, projets, dépôts et membres via l'API 2.0, lire le code source et la configuration du pipeline, et extraire des secrets de l'historique complet des clones manuellement et avec des scanners d'historique.
- **[Abus de Pipelines](pipelines-abuse.md)** : déclencher `bitbucket-pipelines.yml` depuis une branche ou une pull request que vous pouvez pousser, exfiltrer les variables de dépôt et sécurisées en contournant le masquage des journaux, et atteindre les identifiants de déploiement et les rôles cloud négociés via OIDC.
- **[Abus de jetons et de mots de passe d'application](token-and-app-password-abuse.md)** : identifier et réutiliser des mots de passe d'application divulgués, des jetons d'accès de dépôt/projet/workspace, et des jetons API Atlassian contre l'API et `git`, lire leur identité et leur portée, et agir dans leurs limites.

## Références

- [Bitbucket Cloud REST API reference](https://developer.atlassian.com/cloud/bitbucket/rest/intro/)
- [Bitbucket Pipelines documentation](https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/)
- [HackTricks Cloud: Bitbucket Pipelines security](https://cloud.hacktricks.wiki/en/pentesting-ci-cd/bitbucket-security/index.html)
- [Rhino Security Labs: CI/CD pipeline attacks](https://rhinosecuritylabs.com/aws/circleci-attack-aws/)
