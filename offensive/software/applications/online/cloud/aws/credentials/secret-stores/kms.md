---
title: "KMS: Decrypt and permissive key policies"
order: 3
description: "Abusing kms:Decrypt and permissive key policies to decrypt protected data and envelope-encrypted secrets."
keywords:
  - KMS
  - Decrypt
  - key policy
  - envelope encryption
  - data key
---

# KMS

KMS guards the keys that wrap everything else: SecureString parameters, encrypted S3 objects, EBS volumes, and application envelope-encrypted blobs. A principal with `kms:Decrypt` on the right key, or a key whose resource policy is too permissive, turns ciphertext you have exfiltrated into plaintext.

## Decrypting with a held grant

```bash
aws kms list-keys ; aws kms list-aliases
# decrypt a blob (ciphertext from an encrypted parameter, file, or object)
aws kms decrypt --ciphertext-blob fileb://blob.bin \
  --query Plaintext --output text | base64 -d
```

## Reading the key policy

```bash
aws kms get-key-policy --key-id <id> --policy-name default --output text
# a Principal of "*" or a broad account root grants decrypt to more than intended
```

## Exploitation notes

- `kms:Decrypt` is the quiet partner of secret theft: `GetParameter --with-decryption` and encrypted-object reads both depend on it.
- Envelope encryption stores the wrapped data key next to the ciphertext; decrypt the data key with KMS, then decrypt the payload locally.
- A permissive key policy can allow cross-account decrypt, letting a foothold in one account read another's protected data.

## Tools

- **AWS CLI** (`kms decrypt`, `kms get-key-policy`).
- **Pacu** (`kms__enum`): enumerate keys and policies.

## References

- [AWS: KMS Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html)
- [HackTricks Cloud: KMS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-kms-enum.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
