---
title: "Escalade de privilèges"
order: 2
description: "Le catalogue d'escalade de privilèges IAM GCP : attributions resourcemanager setIamPolicy, modifications de rôles personnalisés, assouplissement de l'org policy, injection dans les métadonnées d'instance et les chemins actAs de déploiement sous identité de compte de service."
keywords:
  - privilege escalation
  - setIamPolicy
  - actAs
  - custom role
  - org policy
  - service account
---

# Escalade de privilèges

L'escalade de privilèges dans GCP est un problème de permissions : une seule permission dangereuse transforme un membre limité en propriétaire du projet ou de l'organisation. Le catalogue ci-dessous s'appuie sur la liste de méthodes de Rhino Security Labs et sur le projet GCP-IAM-Privilege-Escalation, regroupé selon la primitive que chaque chemin exploite.

## Les chemins

- **[setIamPolicy](set-iam-policy.md)** : vous accorder n'importe quel rôle sur une ressource, un projet, un dossier ou une organisation que vous pouvez administrer.
- **[Mise à jour de rôle personnalisé](custom-role-update.md)** : ajouter des permissions à un rôle personnalisé que vous détenez déjà avec `iam.roles.update`.
- **[Org policy](org-policy.md)** : assouplir les contraintes de l'organization policy avec `orgpolicy.policy.set` pour débloquer d'autres escalades.
- **[Script de démarrage via métadonnées](metadata-startup-script.md)** : exécuter du code sur une instance existante en écrivant `compute.instances.setMetadata` ou une clé SSH.
- **[Déploiement actAs](act-as-deployment/index.md)** : `iam.serviceAccounts.actAs` plus un verbe de création de ressource pour déployer du code qui s'exécute sous l'identité d'un compte de service privilégié.

## Choisir un chemin

Le chemin le plus direct est `setIamPolicy` si vous le détenez : il accorde Owner en un seul appel. Lorsque vous ne détenez que `actAs` plus une permission de déploiement, choisissez la ressource la moins coûteuse à créer (une fonction ou un Cloud Build est plus rapide et plus discret qu'une VM). Les primitives d'usurpation de compte de service (`getAccessToken`, création de clé) se trouvent sous [usurpation de compte de service](../service-account-impersonation/index.md) et s'enchaînent souvent après l'un de ces chemins.

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [GCP IAM privilege escalation (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [HackTricks Cloud : escalade de privilèges GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/gcp-privilege-escalation/index.html)
