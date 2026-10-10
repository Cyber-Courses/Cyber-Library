---
title: "KMS : Decrypt et stratégies de clés permissives"
order: 3
description: "Abuser de kms:Decrypt et de stratégies de clés permissives pour déchiffrer des données protégées et des secrets chiffrés par enveloppe."
keywords:
  - KMS
  - Decrypt
  - stratégie de clé
  - chiffrement par enveloppe
  - clé de données
---

# KMS

KMS protège les clés qui enveloppent tout le reste : paramètres SecureString, objets S3 chiffrés, volumes EBS et blobs applicatifs chiffrés par enveloppe. Un principal disposant de `kms:Decrypt` sur la bonne clé, ou une clé dont la stratégie de ressource est trop permissive, transforme en clair le texte chiffré que vous avez exfiltré.

## Déchiffrer avec une autorisation détenue

```bash
aws kms list-keys ; aws kms list-aliases
# decrypt a blob (ciphertext from an encrypted parameter, file, or object)
aws kms decrypt --ciphertext-blob fileb://blob.bin \
  --query Plaintext --output text | base64 -d
```

## Lire la stratégie de clé

```bash
aws kms get-key-policy --key-id <id> --policy-name default --output text
# a Principal of "*" or a broad account root grants decrypt to more than intended
```

## Notes d'exploitation

- `kms:Decrypt` est le partenaire discret du vol de secrets : `GetParameter --with-decryption` et la lecture d'objets chiffrés en dépendent tous deux.
- Le chiffrement par enveloppe stocke la clé de données enveloppée à côté du texte chiffré ; déchiffrez la clé de données avec KMS, puis déchiffrez la charge utile localement.
- Une stratégie de clé permissive peut autoriser le déchiffrement inter-comptes, laissant un point d'ancrage dans un compte lire les données protégées d'un autre.

## Outils

- **AWS CLI** (`kms decrypt`, `kms get-key-policy`).
- **Pacu** (`kms__enum`) : énumérer les clés et les stratégies.

## Références

- [AWS: KMS Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)
- [HackTricks Cloud: KMS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-kms-enum.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
