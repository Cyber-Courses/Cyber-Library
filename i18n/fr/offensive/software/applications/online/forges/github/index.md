---
title: "GitHub : attaquer les organisations, dépôts, Actions et jetons"
order: 1
description: "Techniques offensives contre GitHub en tant que cible SaaS : reconnaissance d'organisation et de dépôt, collecte de secrets dans l'historique, injection de workflow Actions, abus de GITHUB_TOKEN et de PAT, persistance via les OAuth Apps et les webhooks, et prise de contrôle de runner auto-hébergé."
keywords:
  - GitHub
  - GitHub Actions
  - GITHUB_TOKEN
  - injection de workflow
  - runner auto-hébergé
---

# GitHub

GitHub est une forge de code atteinte entièrement via ses **API REST et GraphQL** (base `https://api.github.com`) et le protocole `git`. Presque chaque attaque est une question de **ce qu'un jeton peut voir et faire**, et de **ce qui s'exécute lorsqu'un code ou un événement entre dans un dépôt**. La chaîne est constante : énumérer l'organisation et ses dépôts, récolter des secrets depuis les clones et l'historique, transformer un pipeline CI en exécution de code, puis abuser des jetons que cette exécution expose pour pousser, déployer et pivoter vers les comptes cloud connectés.

Ces pages visent `github.com` (SaaS). Lorsqu'un GitHub Enterprise Server (GHES) auto-hébergé se comporte différemment, cela est signalé en ligne.

## Triage

Établissez l'identité que vous détenez et la surface organisation/dépôt avant toute autre chose :

```bash
# What does this token see, and what scopes does it carry
curl -sI -H "Authorization: token $GH_TOKEN" https://api.github.com/user | grep -i '^x-oauth-scopes:'
gh auth status

# Org inventory: members, repos (public + private the token can see), teams
gh api /orgs/ACME/repos --paginate -q '.[].full_name'
gh api /orgs/ACME/members --paginate -q '.[].login'
gh api /user/repos --paginate -q '.[] | select(.permissions.admin==true) | .full_name'
```

Un jeton qui renvoie des dépôts privés, ou une ligne `X-OAuth-Scopes` contenant `repo`, `workflow` ou `admin:org`, est déjà une position forte. Les dépôts avec un répertoire `.github/workflows/` constituent la surface d'attaque CI ; les dépôts utilisant `pull_request_target`, `workflow_run` ou `runs-on: self-hosted` sont les plus précieux.

## Pages

- **[Reconnaissance](reconnaissance.md)** : énumérer les organisations, dépôts, membres et forks via REST et GraphQL, la recherche de code GitHub, les gists, et récolter des secrets depuis tout l'historique de clonage et les objets non référencés, manuellement et avec TruffleHog et Gitleaks.
- **[Injection de workflow Actions](actions-workflow-injection.md)** : abuser de `pull_request_target` et `workflow_run` pour exécuter le code de PR de l'attaquant avec un `GITHUB_TOKEN` privilégié et des secrets, et l'injection de script shell depuis des expressions de contexte non fiables, avec exfiltration de secrets, exécution de pipeline empoisonnée et empoisonnement du cache.
- **[Abus des jetons et de GITHUB_TOKEN](token-and-github-token-abuse.md)** : permissions par défaut de `GITHUB_TOKEN` et cadrage par `permissions:`, la portée `workflow`, l'usage de PAT classiques et fine-grained divulgués, la lecture des portées depuis les en-têtes de réponse, et l'échange d'un jeton OIDC contre un rôle cloud.
- **[Abus des OAuth Apps et des webhooks](oauth-app-and-webhook-abuse.md)** : persistance via les OAuth Apps autorisées et les GitHub Apps installées avec des jetons d'accès d'installation, le phishing de consentement, et les webhooks de dépôt/organisation qui exfiltrent les charges utiles d'événements.
- **[Prise de contrôle de runner auto-hébergé](self-hosted-runner-takeover.md)** : obtenir du RCE sur des runners auto-hébergés non éphémères via les workflows de PR, enregistrer des runners pirates avec un jeton divulgué, piller les répertoires de travail et d'identifiants du runner, et pivoter vers le réseau interne.

## Références

- [GitHub REST API documentation](https://docs.github.com/en/rest)
- [Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions)
- [PortSwigger Research : GitHub Actions attack diary](https://www.synacktiv.com/en/publications/github-actions-exploitation-introduction.html)
- [Praetorian gato : GitHub Actions attack tooling](https://github.com/praetorian-inc/gato)
- [PayloadsAllTheThings : CI/CD](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/CI%20CD/README.md)
