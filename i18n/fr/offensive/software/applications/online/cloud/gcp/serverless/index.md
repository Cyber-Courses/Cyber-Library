---
title: "GCP sans serveur"
order: 5
description: "Attaquer les services sans serveur et de CI de GCP : comptes de service de Cloud Functions et Cloud Run, et abus du compte de service et des déclencheurs de Cloud Build."
keywords:
  - Cloud Functions
  - Cloud Run
  - Cloud Build
  - compte de service
  - CI/CD
---

# Sans serveur

Les services sans serveur et de CI de GCP s'exécutent chacun avec un **compte de service**, ce qui en fait à la fois une cible de butin (lire leur code source, leur environnement et leurs secrets) et une surface d'exécution (déployer ou déclencher du code qui s'exécute avec un compte privilégié). Cloud Build se distingue particulièrement : son compte de service par défaut dispose historiquement du rôle **Editor** au niveau du projet, de sorte qu'un simple build que vous contrôlez permet d'obtenir un jeton quasi administrateur.

L'angle d'élévation de privilèges par déploiement avec un compte de service (`iam.serviceAccounts.actAs` associé à un verbe de création) est répertorié sous [élévation de privilèges d'identité](../identity/privilege-escalation/index.md) ; les pages ici couvrent l'attaque des services eux-mêmes et leur utilisation pour la persistance.

## Pages

- **[Cloud Functions](cloud-functions.md)** : lecture du code source et de l'environnement, invocation non authentifiée, et compte de service d'exécution.
- **[Cloud Run](cloud-run.md)** : services et jobs non authentifiés, secrets de révision, et compte de service attaché.
- **[Cloud Build](cloud-build/index.md)** : exfiltration du jeton du compte de service de build et mise en place de déclencheurs de build pour la persistance.

## Références

- [HackTricks Cloud: GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Rhino Security Labs: GCP privilege escalation](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
