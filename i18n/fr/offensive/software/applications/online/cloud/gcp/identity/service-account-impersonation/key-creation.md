---
title: "Création de clé : générer une clé à longue durée de vie pour un compte de service"
order: 3
description: "Créer une clé JSON à longue durée de vie pour un compte de service cible avec iam.serviceAccountKeys.create pour un accès durable et de la persistance."
keywords:
  - serviceAccountKeys.create
  - clé de compte de service
  - clé JSON
  - persistance
  - usurpation d'identité
---

# Création de clé

`iam.serviceAccountKeys.create` génère une clé JSON gérée par l'utilisateur pour un compte de service cible. Contrairement à un jeton d'usurpation à courte durée de vie, la clé est à longue durée de vie et portable : elle constitue à la fois une prise de contrôle immédiate du compte et une persistance durable qui survit aux changements de mot de passe et de session.

## Créer et activer une clé

```bash
gcloud iam service-accounts keys create key.json \
  --iam-account <target>@<proj>.iam.gserviceaccount.com

# authenticate as the target from anywhere
gcloud auth activate-service-account --key-file key.json
gcloud auth print-access-token
```

## Notes d'exploitation

- La clé n'expire jamais par défaut, c'est donc un mécanisme de persistance de prédilection : exfiltrez `key.json` et revenez à volonté.
- La création de clé sur un compte de service privilégié est une escalade complète ; énumérez pour quels comptes vous pouvez créer des clés.
- Plus bruyant que [getAccessToken](get-access-token.md) et [signJwt](sign-jwt-and-sign-blob.md) car cela laisse un objet clé répertorié sur le compte ; préférez ces méthodes lorsque vous n'avez besoin que d'un accès transitoire.

## Outils

- **gcloud** (`iam service-accounts keys create`).

## Références

- [Rhino Security Labs : escalade de privilèges GCP (partie 1)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Escalade de privilèges GCP IAM (dépôt Rhino)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google : gestion des clés de compte de service](https://cloud.google.com/iam/docs/keys-create-delete)
