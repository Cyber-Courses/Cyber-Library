---
title: "Identité GCP"
order: 1
description: "Attaquer Google Cloud IAM : énumérer les membres et les rôles, le catalogue d'escalade de privilèges (setIamPolicy, déploiements actAs, métadonnées), l'impersonation de comptes de service et la workload identity federation."
keywords:
  - Cloud IAM
  - service account
  - privilege escalation
  - impersonation
  - actAs
---

# Identité

Cloud IAM est le terrain où se gagnent les attaques GCP. Une liaison accorde à un membre (utilisateur, groupe ou **compte de service**) un rôle à une portée, et tout le jeu consiste à transformer une permission que vous détenez en contrôle d'un compte de service plus privilégié. Deux mécanismes dominent : l'**impersonation**, où une permission comme `iam.serviceAccounts.getAccessToken` génère directement le token d'un autre compte, et **actAs**, où `iam.serviceAccounts.actAs` plus une permission de création de ressource vous permet de déployer du code (une VM, une fonction, un build ou un job) qui s'exécute sous un compte privilégié.

L'escalade de privilèges se trouve ici parce que dans GCP c'est un problème de permissions : une seule permission dangereuse (`resourcemanager.projects.setIamPolicy`, `iam.serviceAccountKeys.create`, `iam.serviceAccounts.actAs` avec un verbe de déploiement) promeut un membre limité au rang de propriétaire du projet ou de l'organisation.

## Ce qui est regroupé ici

- **[Énumération](enumeration.md)** : résoudre les membres, les rôles et les comptes de service que vous pouvez atteindre, avec `gcloud` et les outils de graphe IAM.
- **[Escalade de privilèges](privilege-escalation/index.md)** : `setIamPolicy`, mises à jour de rôles personnalisés, politique d'organisation, scripts de démarrage via métadonnées et le catalogue `actAs` de déploiement-en-tant-que-compte-de-service.
- **[Impersonation de compte de service](service-account-impersonation/index.md)** : `getAccessToken`, `signJwt` et `signBlob`, octrois Token Creator, délégation implicite et création de clé.
- **[Fédération](federation.md)** : workload identity federation depuis une identité OIDC externe ou AWS vers un compte de service GCP.

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [GCP IAM privilege escalation (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [HackTricks Cloud: GCP privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/gcp-privilege-escalation/index.html)
