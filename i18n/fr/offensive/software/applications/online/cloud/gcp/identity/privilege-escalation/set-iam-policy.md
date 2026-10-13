---
title: "setIamPolicy : vous accorder n'importe quel rôle à un scope donné"
description: "Vous accorder n'importe quel rôle en appelant resourcemanager.projects, folders ou organizations setIamPolicy sur une ressource que vous pouvez administrer."
keywords:
  - setIamPolicy
  - liaison IAM
  - octroi de rôle
  - resourcemanager
  - owner
---

# setIamPolicy

`setIamPolicy` est la capacité de réécrire la politique IAM d'une ressource. Si vous détenez `resourcemanager.projects.setIamPolicy` (ou la variante dossier ou organisation, ou un `*.setIamPolicy` au niveau de la ressource), vous ajoutez une liaison vous accordant `roles/owner` et l'escalade est terminée en un seul appel. De nombreux rôles prédéfinis portent une permission `setIamPolicy` pour leur service sans que cela soit évident.

## Vous accorder Owner sur le projet

```bash
gcloud projects add-iam-policy-binding <project> \
  --member="user:you@example.com" --role="roles/owner"
# or for a controlled service account
gcloud projects add-iam-policy-binding <project> \
  --member="serviceAccount:<sa>@<project>.iam.gserviceaccount.com" --role="roles/owner"
```

Au niveau d'un dossier ou d'une organisation, le même mouvement accorde le contrôle de chaque projet en dessous :

```bash
gcloud resource-manager folders add-iam-policy-binding <folder-id> --member=... --role="roles/owner"
gcloud organizations add-iam-policy-binding <org-id> --member=... --role="roles/resourcemanager.organizationAdmin"
```

## Variantes au niveau de la ressource

Lorsque vous ne détenez `setIamPolicy` que sur un seul type de ressource, accordez-vous un rôle sur cette ressource et pivotez à partir d'elle (par exemple `iam.serviceAccounts.setIamPolicy` sur un compte de service accorde Token Creator, couvert dans [Octroi de Token Creator](../service-account-impersonation/token-creator-grant.md)).

## Notes d'exploitation

- `add-iam-policy-binding` lit puis écrit la politique ; une concurrence avec une modification simultanée peut faire disparaître votre liaison, relisez donc pour confirmer qu'elle a bien été appliquée.
- Owner à un scope inférieur suffit pour la plupart des objectifs ; obtenir l'administration au niveau de l'organisation est plus bruyant et souvent inutile.
- La liaison est visible dans la politique IAM de la ressource et dans les journaux d'audit, elle ne sert donc de persistance que jusqu'à ce que quelqu'un examine les liaisons.

## Outils

- **gcloud** (`add-iam-policy-binding`, `set-iam-policy`) : l'octroi.
- **GCP-IAM-Privilege-Escalation** (Rhino) : détecte quelles permissions `setIamPolicy` le membre courant détient.

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Escalade de privilèges GCP IAM (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google : accorder, modifier et révoquer l'accès](https://cloud.google.com/iam/docs/granting-changing-revoking-access)
