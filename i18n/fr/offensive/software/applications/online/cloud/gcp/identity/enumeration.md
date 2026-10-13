---
title: "Énumération : cartographier le graphe IAM et les comptes de service accessibles"
order: 1
description: "Cartographier les liaisons IAM, les membres, les rôles personnalisés et les comptes de service d'un projet GCP depuis un point d'ancrage : appels getIamPolicy, gcloud et des outils comme ScoutSuite pour trouver les identités et permissions accessibles."
keywords:
  - IAM enumeration
  - getIamPolicy
  - gcloud
  - service accounts
  - custom roles
  - ScoutSuite
---

# Énumération

Toute escalade dans GCP commence par le graphe IAM : quels membres détiennent quels rôles à quelle portée, et quels comptes de service vous pouvez atteindre par impersonation ou `actAs`. Les réponses viennent des appels `getIamPolicy` à chaque niveau et de la résolution des permissions que détient votre propre identité.

## Qui suis-je et que détiens-je

```bash
gcloud auth list                                    # active identities
gcloud config get-value project
gcloud projects get-iam-policy <project> --format=json   # all bindings at the project
# test a specific permission set against the project
gcloud iam list-testable-permissions //cloudresourcemanager.googleapis.com/projects/<project>
```

## Lire les liaisons à travers les portées

```bash
gcloud organizations get-iam-policy <org-id>
gcloud resource-manager folders get-iam-policy <folder-id>
gcloud projects get-iam-policy <project>
# service accounts you might impersonate or actAs
gcloud iam service-accounts list
gcloud iam service-accounts get-iam-policy <sa-email>   # who can mint its token
```

## Résoudre les chemins accessibles

```bash
# asset inventory for the whole resource graph
gcloud asset search-all-resources --scope=projects/<project>
gcloud asset search-all-iam-policies --scope=projects/<project>
```

ScoutSuite et les scripts de graphe IAM du dépôt Rhino GCP-IAM-Privilege-Escalation transforment ce vidage en une liste d'arêtes d'escalade : qui peut faire `setIamPolicy`, qui détient `actAs` plus un verbe de déploiement, et quels comptes de service ont Token Creator lié à un principal que vous contrôlez.

## Notes d'exploitation

- `get-iam-policy` à une portée ne renvoie que les liaisons définies à cette portée ; les rôles hérités d'un dossier parent ou de l'organisation se résolvent avec la recherche de politiques de l'asset-inventory, et non avec la seule politique du projet.
- Un `getIamPolicy` refusé ne signifie pas que vous êtes bloqué : `list-testable-permissions` et le simple fait de tenter les appels d'escalade révèlent la surface exploitable.
- Les comptes de service par défaut de Compute Engine et App Engine détiennent fréquemment le rôle Editor, donc notez chaque ressource qui s'exécute sous l'un d'eux.

## Outils

- **gcloud** (`get-iam-policy`, `asset search-all-iam-policies`, `list-testable-permissions`) : le vidage natif du graphe.
- **ScoutSuite** : posture multi-services et inventaire IAM.
- **GCP-IAM-Privilege-Escalation** (Rhino) : scripts qui énumèrent les permissions et signalent les chemins d'escalade.

## Références

- [GCP IAM privilege escalation (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google : comprendre les politiques IAM](https://cloud.google.com/iam/docs/policies)
