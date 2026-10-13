---
title: "Parameter Store: GetParameter over SecureString and plaintext"
order: 2
description: "ssm:GetParameter and GetParameters to read SecureString and plaintext parameters holding credentials and configuration."
keywords:
  - Parameter Store
  - SSM
  - SecureString
  - GetParameter
  - config
---

# Parameter Store

SSM Parameter Store holds configuration and secrets as named parameters, `SecureString` values KMS-encrypted and the rest in plaintext. `ssm:GetParameter` with `--with-decryption` (plus `kms:Decrypt` on the key) returns the cleartext, so applications and attackers read it the same way.

## Listing and reading

```bash
aws ssm describe-parameters --query 'Parameters[].Name' --output text
aws ssm get-parameter --name <name> --with-decryption \
  --query Parameter.Value --output text
# recurse a path prefix, decrypting as you go
aws ssm get-parameters-by-path --path / --recursive --with-decryption \
  --query 'Parameters[].[Name,Value]' --output text
```

## Exploitation notes

- `--with-decryption` needs `kms:Decrypt` on the backing key as well as the SSM read; a denial there means you hold SSM but not the KMS grant, see [KMS](kms.md).
- Plaintext (`String`) parameters need no KMS permission at all and often still hold credentials put there carelessly.
- `get-parameters-by-path --recursive` over `/` is the fastest full sweep when the naming is hierarchical.

## Tools

- **AWS CLI** (`ssm get-parameters-by-path --with-decryption`).
- **Pacu** (`secrets__enum`): bulk extraction.

## References

- [AWS: SSM GetParameter](https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_GetParameter.html)
- [HackTricks Cloud: SSM Parameter Store](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ssm-enum.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
