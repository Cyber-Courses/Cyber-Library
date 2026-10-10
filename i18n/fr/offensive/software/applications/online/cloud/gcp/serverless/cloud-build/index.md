---
title: "Cloud Build"
order: 3
description: "Abuser de Cloud Build comme surface d'élévation de privilèges et de persistance : les rôles étendus du compte de service de build et les déclencheurs de build contrôlés par l'attaquant."
keywords:
  - Cloud Build
  - compte de service de build
  - déclencheurs
  - CI/CD
  - élévation de privilèges
---

# Cloud Build

Cloud Build exécute chaque build en tant que **compte de service** Cloud Build (`<project-number>@cloudbuild.gserviceaccount.com`), qui, par défaut, s'est historiquement vu accorder le rôle **Editor** du projet. Quiconque peut lancer un build, ou déposer un déclencheur qui en lance un, exécute donc des étapes arbitraires en tant que principal quasi administrateur. Cela fait de Cloud Build l'une des plus fortes surfaces d'élévation de privilèges et de persistance de GCP.

## Pages

- **[Compte de service de build](build-service-account.md)** : lancer un build dont les étapes exfiltrent le jeton du compte de service de build et agissent avec ses rôles.
- **[Déclencheurs de build](build-triggers.md)** : déposer ou modifier des déclencheurs de dépôt pour que les pushs exécutent des étapes de l'attaquant en tant que compte de service de build.

## Références

- [Rhino Security Labs : élévation de privilèges GCP (Cloud Build)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [HackTricks Cloud : GCP Cloud Build](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Élévation de privilèges IAM GCP (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
