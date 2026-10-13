---
title: "Parameter Store : GetParameter sur SecureString et valeurs en clair"
order: 2
description: "ssm:GetParameter et GetParameters pour lire les paramètres SecureString et en clair contenant identifiants et configuration."
keywords:
  - Parameter Store
  - SSM
  - SecureString
  - GetParameter
  - config
---

# Parameter Store

SSM Parameter Store conserve la configuration et les secrets sous forme de paramètres nommés, les valeurs `SecureString` chiffrées par KMS et le reste en clair. `ssm:GetParameter` avec `--with-decryption` (plus `kms:Decrypt` sur la clé) renvoie le texte en clair, de sorte qu'applications et attaquants le lisent de la même façon.

## Lister et lire

```bash
aws ssm describe-parameters --query 'Parameters[].Name' --output text
aws ssm get-parameter --name <name> --with-decryption \
  --query Parameter.Value --output text
# recurse a path prefix, decrypting as you go
aws ssm get-parameters-by-path --path / --recursive --with-decryption \
  --query 'Parameters[].[Name,Value]' --output text
```

## Notes d'exploitation

- `--with-decryption` nécessite `kms:Decrypt` sur la clé sous-jacente en plus de la lecture SSM ; un refus à ce niveau signifie que vous détenez SSM mais pas l'autorisation KMS, voir [KMS](kms.md).
- Les paramètres en clair (`String`) ne requièrent aucune permission KMS et contiennent souvent tout de même des identifiants déposés là par négligence.
- `get-parameters-by-path --recursive` sur `/` est le balayage complet le plus rapide quand la nomenclature est hiérarchique.

## Outils

- **AWS CLI** (`ssm get-parameters-by-path --with-decryption`).
- **Pacu** (`secrets__enum`) : extraction en masse.

## Références

- [AWS: SSM GetParameter](https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_GetParameter.html)
- [HackTricks Cloud: SSM Parameter Store](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ssm-enum.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
