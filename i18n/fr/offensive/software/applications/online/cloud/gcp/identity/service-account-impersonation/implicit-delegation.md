---
title: "Délégation implicite : enchaîner l'usurpation d'identité via un intermédiaire"
order: 2
description: "Enchaîner l'usurpation d'identité via un compte de service intermédiaire avec iam.serviceAccounts.implicitDelegation pour atteindre une cible que vous ne pouvez pas appeler directement."
keywords:
  - délégation implicite
  - chaîne de délégation
  - compte de service
  - usurpation d'identité
  - getAccessToken
---

# Délégation implicite

Lorsque vous ne pouvez pas usurper directement un compte de service cible mais que vous pouvez usurper un compte intermédiaire qui détient Token Creator sur la cible, la **délégation** passe par lui. Vous demandez à IAM Credentials de générer le jeton de la cible tout en déléguant via l'intermédiaire, atteignant un compte à un saut de distance.

## Enchaîner via un délégué

```bash
# you can impersonate SA-A; SA-A has Token Creator on SA-B (the real target)
gcloud auth print-access-token \
  --impersonate-service-account <sa-b>@<proj>.iam.gserviceaccount.com \
  --delegates <sa-a>@<proj>.iam.gserviceaccount.com
```

L'API `generateAccessToken` accepte une liste `delegates`, donc des chaînes plus longues (A vers B vers C) fonctionnent tant que chaque maillon détient Token Creator sur le suivant.

## Notes d'exploitation

- Cartographiez le graphe Token Creator à partir des politiques IAM : la délégation transforme une chaîne d'octrois faibles en un chemin vers un compte puissant.
- La délégation est facile à manquer lors d'une revue, car aucune liaison unique ne vous nomme sur la cible finale.
- Chaque saut est un jeton à courte durée de vie, régénérez donc au fur et à mesure du parcours de la chaîne.

## Outils

- **gcloud** (`--impersonate-service-account` avec `--delegates`).
- Analyse de graphe IAM pour trouver les chaînes de délégation.

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [Escalade de privilèges GCP IAM (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google : identifiants délégués](https://cloud.google.com/iam/docs/create-short-lived-credentials-delegated)
