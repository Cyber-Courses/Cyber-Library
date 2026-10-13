---
title: "Applications Entra"
order: 2
description: "Abus des enregistrements d'applications Entra et des principaux de service : hameçonnage par consentement, abus d'autorisations Microsoft Graph, identifiants ajoutés, identifiants d'identité fédérée et prise de contrôle par la propriété."
keywords:
  - enregistrement d'application
  - principal de service
  - hameçonnage par consentement
  - API Graph
  - identifiants d'application
---

# Applications

Les enregistrements d'applications et leurs instances dans le locataire, les **principaux de service**, sont des identités de premier plan dans Entra, et ils sont couramment surprivilégiés. Le contrôle d'une application détenant une autorisation Microsoft Graph de grande valeur équivaut à détenir cette autorisation vous-même, ce qui fait des applications une surface majeure d'élévation de privilèges et de persistance.

## Ce qui est regroupé ici

- **[Principaux de service](service-principals.md)** : énumération et détournement des identités d'application et de leurs rôles d'application.
- **[Hameçonnage par consentement](consent-phishing.md)** : amener des utilisateurs ou des administrateurs à accorder l'accès à une application OAuth malveillante.
- **[Autorisations d'API](api-permissions.md)** : abus des autorisations d'application Microsoft Graph pour atteindre le contrôle du locataire.
- **[Identifiants d'application](application-credentials.md)** : ajout d'un secret ou d'un certificat pour s'authentifier en tant qu'application.
- **[Identifiants fédérés](federated-credentials.md)** : ajout d'un identifiant d'identité fédérée pour forger des jetons sans secret.
- **[Propriété](ownership.md)** : abus de la propriété d'une application ou d'un principal de service pour en prendre le contrôle.

## Références

- [SpecterOps : BARK (chemins d'attaque des applications Azure)](https://github.com/BloodHoundAD/BARK)
- [GraphRunner (dafthack)](https://github.com/dafthack/GraphRunner)
- [HackTricks Cloud : applications Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
