---
title: "Mise à jour de rôle personnalisé : ajouter des permissions à un rôle que vous détenez déjà"
order: 1
description: "Ajouter des permissions à un rôle personnalisé auquel vous êtes déjà lié avec iam.roles.update, pour escalader sans créer de nouvelle liaison."
keywords:
  - iam.roles.update
  - custom role
  - permission
  - privilege escalation
  - role edit
---

# Mise à jour de rôle personnalisé

Si vous êtes lié à un **rôle personnalisé** et que vous détenez `iam.roles.update` sur celui-ci, vous pouvez ajouter n'importe quelle permission à ce rôle et en hériter instantanément, sans aucune nouvelle liaison à créer. C'est discret, car votre appartenance ne change pas : seule la définition d'un rôle que vous possédez déjà est modifiée.

## Ajouter des permissions au rôle

```bash
# see the role you are bound to
gcloud iam roles describe <roleId> --project=<project>

# add the permissions you want (for example the setIamPolicy to then grant Owner)
gcloud iam roles update <roleId> --project=<project> \
  --add-permissions=resourcemanager.projects.setIamPolicy,iam.serviceAccounts.actAs
```

Une fois `resourcemanager.projects.setIamPolicy` présent dans votre rôle, suivez le chemin [setIamPolicy](set-iam-policy.md) vers Owner ; avec `iam.serviceAccounts.actAs`, suivez un [déploiement actAs](act-as-deployment/index.md).

## Notes d'exploitation

- Cela ne fonctionne que sur les rôles **personnalisés** ; les rôles prédéfinis Google ne peuvent pas être modifiés.
- Vous devez déjà être membre du rôle pour que la nouvelle permission vous parvienne ; sinon, la mise à jour plus une auto-liaison représentent deux étapes.
- Ajouter une seule permission pivot (`setIamPolicy` ou `actAs`) est plus furtif que d'accorder tous les droits au rôle.

## Outils

- **gcloud** (`iam roles update --add-permissions`) : la modification.
- **GCP-IAM-Privilege-Escalation** (Rhino) : signale les rôles personnalisés modifiables auxquels vous êtes lié.

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [GCP IAM privilege escalation (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google : création et gestion de rôles personnalisés](https://cloud.google.com/iam/docs/creating-custom-roles)
