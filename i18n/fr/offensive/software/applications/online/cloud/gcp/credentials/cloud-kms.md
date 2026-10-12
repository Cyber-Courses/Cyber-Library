---
title: "Cloud KMS : déchiffrer et signer avec les permissions de clé"
order: 5
description: "Abuser des permissions Cloud KMS pour déchiffrer des données ou signer en tant que clé avec cloudkms.cryptoKeyVersions.useToDecrypt et useToSign."
keywords:
  - Cloud KMS
  - cryptoKeyVersions
  - déchiffrement
  - signature asymétrique
  - accès aux clés
  - chiffrement
---

# Cloud KMS

Cloud KMS détient les clés qui protègent les autres secrets : la permission d'*utiliser* une clé est donc la permission de lire ce qu'elle protège. `cloudkms.cryptoKeyVersions.useToDecrypt` déchiffre tout texte chiffré enveloppé avec la clé (secrets d'application, blobs chiffrés par enveloppe, données stockées), et `useToSign` permet de forger des signatures en tant que clé asymétrique, ce qui peut servir à générer des jetons ou à signer des artefacts de confiance en aval.

## Déchiffrement

```bash
gcloud kms keys list --location=<loc> --keyring=<ring>
gcloud kms decrypt --location=<loc> --keyring=<ring> --key=<key> \
  --ciphertext-file=blob.enc --plaintext-file=blob.txt
```

## Signer en tant que clé asymétrique

```bash
gcloud kms asymmetric-signature sign --location=<loc> --keyring=<ring> \
  --key=<key> --version=1 --digest-algorithm=sha256 \
  --input-file=payload --signature-file=payload.sig
```

## Notes d'exploitation

- Les permissions IAM de KMS sont souvent définies au niveau du trousseau ou de la clé, si bien qu'un principal sans rôle KMS au niveau du projet peut tout de même détenir `useToDecrypt` sur une clé donnée : il faut énumérer clé par clé.
- L'accès en déchiffrement transforme n'importe quel coffre de secrets chiffré par enveloppe ou sauvegarde en texte clair, sans même passer par le service de données qui l'a écrit.
- Signer avec une clé de confiance peut permettre de forger des JWT ou des signatures d'artefacts acceptées par d'autres systèmes, étendant la portée au-delà de GCP.

## Outils

- **gcloud** (`kms decrypt`, `kms asymmetric-signature sign`).

## Références

- [Google : Cloud KMS decrypt](https://cloud.google.com/kms/docs/encrypt-decrypt)
- [HackTricks Cloud : GCP KMS](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
