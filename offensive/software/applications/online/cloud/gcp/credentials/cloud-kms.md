---
title: "Cloud KMS: decrypting and signing with key permissions"
order: 5
description: "Abusing Cloud KMS permissions to decrypt data or sign as a key with cloudkms.cryptoKeyVersions.useToDecrypt and useToSign."
keywords:
  - Cloud KMS
  - cryptoKeyVersions
  - decrypt
  - asymmetric sign
  - key access
  - encryption
---

# Cloud KMS

Cloud KMS holds the keys that protect other secrets, so permission to *use* a key is permission to read what it protects. `cloudkms.cryptoKeyVersions.useToDecrypt` decrypts any ciphertext wrapped with the key (application secrets, envelope-encrypted blobs, stored data), and `useToSign` forges signatures as an asymmetric key, which can mint tokens or sign artifacts trusted downstream.

## Decrypting

```bash
gcloud kms keys list --location=<loc> --keyring=<ring>
gcloud kms decrypt --location=<loc> --keyring=<ring> --key=<key> \
  --ciphertext-file=blob.enc --plaintext-file=blob.txt
```

## Signing as an asymmetric key

```bash
gcloud kms asymmetric-signature sign --location=<loc> --keyring=<ring> \
  --key=<key> --version=1 --digest-algorithm=sha256 \
  --input-file=payload --signature-file=payload.sig
```

## Exploitation notes

- KMS IAM is often set at the keyring or key level, so a principal with no project-wide KMS role may still hold `useToDecrypt` on one key; enumerate per key.
- Decrypt access turns any envelope-encrypted secret store or backup into cleartext without touching the data service that wrote it.
- Signing with a trusted key can forge JWTs or artifact signatures that other systems accept, extending reach beyond GCP.

## Tools

- **gcloud** (`kms decrypt`, `kms asymmetric-signature sign`).

## References

- [Google: Cloud KMS decrypt](https://cloud.google.com/kms/docs/encrypt-decrypt)
- [HackTricks Cloud: GCP KMS](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
