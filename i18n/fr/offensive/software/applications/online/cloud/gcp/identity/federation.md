---
title: "Fédération : Workload Identity Federation vers un compte de service"
order: 4
description: "Entrer dans GCP via Workload Identity Federation : échanger un token OIDC externe ou AWS et Azure contre un token de compte de service Google sans clé."
keywords:
  - workload identity federation
  - OIDC
  - token exchange
  - service account
  - external identity
---

# Fédération

Workload Identity Federation permet à une identité externe (un émetteur OIDC, un rôle AWS ou une identité Azure) d'impersonner un **compte de service** Google sans clé à longue durée de vie. Un workload identity pool déclare l'émetteur de confiance et un mappage d'attributs ; un compte de service accorde aux principaux du pool le rôle `roles/iam.workloadIdentityUser`. Lorsque la condition d'attribut est trop permissive, un attaquant capable d'obtenir un token externe correspondant l'échange contre le token Google du compte de service.

## Échanger un token externe

```bash
# the external credential config points at the pool provider and the token source
gcloud iam workload-identity-pools create-cred-config \
  projects/<n>/locations/global/workloadIdentityPools/<pool>/providers/<prov> \
  --service-account=<target-sa>@<project>.iam.gserviceaccount.com \
  --output-file=cred.json --credential-source-file=token.jwt
GOOGLE_APPLICATION_CREDENTIALS=cred.json gcloud auth print-access-token --impersonate-service-account=<target-sa>
```

Le point de terminaison STS `sts.googleapis.com/v1/token` réalise l'échange brut si vous le pilotez directement avec le JWT externe.

## Lire la relation de confiance

```bash
gcloud iam workload-identity-pools providers describe <prov> \
  --workload-identity-pool=<pool> --location=global
# inspect attribute-mapping and attribute-condition: a missing or wildcard
# condition on google.subject / attribute.repository accepts any issuer token
```

## Notes d'exploitation

- La faiblesse est l'`attribute-condition` du provider : aucune condition, ou une qui vérifie seulement l'émetteur et non le sujet, laisse n'importe quel token émis par l'émetteur assumer le compte de service.
- GitHub Actions, GitLab et les providers OIDC génériques sont les émetteurs courants ; un mappage `attribute.repository` trop permissif accepte les workflows provenant de dépôts que vous contrôlez.
- La fédération ne laisse aucune clé derrière elle, c'est donc aussi un point d'ancrage de persistance lorsque vous pouvez créer vous-même le pool et la liaison.

## Outils

- **gcloud** (`iam workload-identity-pools`, `auth print-access-token --impersonate-service-account`) : configurer et exercer l'échange.
- **jwt_tool** : inspecter et façonner les claims du JWT externe.

## Références

- [Rhino Security Labs : escalade de privilèges GCP](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google : Workload Identity Federation](https://cloud.google.com/iam/docs/workload-identity-federation)
