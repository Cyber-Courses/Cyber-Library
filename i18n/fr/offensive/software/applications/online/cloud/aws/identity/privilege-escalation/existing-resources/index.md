---
title: "Ressources existantes"
order: 5
description: "Escalader via des ressources qui s'exécutent déjà avec un rôle privilégié : mise à jour de stacks CloudFormation, de dev endpoints Glue et de code Lambda, ou génération d'URL présignées et d'accès EC2 Instance Connect."
keywords:
  - UpdateStack
  - UpdateFunctionCode
  - dev endpoint
  - URL présignée
  - EC2 Instance Connect
  - ressource existante
---

# Ressources existantes

Lorsqu'une ressource s'exécute déjà avec un rôle privilégié, vous n'avez pas besoin de `iam:PassRole` : il vous suffit d'avoir l'autorisation de **modifier ce que cette ressource exécute**. Écraser son code ou sa configuration fait exécuter votre charge utile par le rôle existant.

## Chemins

- **[UpdateStack](update-stack.md)** : modifier une stack CloudFormation qui porte un rôle de stack privilégié.
- **[UpdateDevEndpoint](update-dev-endpoint.md)** : pousser une clé SSH vers un dev endpoint Glue exécutant un rôle privilégié.
- **[UpdateFunctionCode](update-function-code.md)** : écraser une Lambda qui exécute déjà un rôle d'exécution privilégié.
- **[URL présignée](presigned-url.md)** : générer un accès signé à durée limitée à des ressources sous vos propres autorisations.
- **[Instance Connect](instance-connect.md)** : pousser une clé SSH temporaire vers une instance portant un profil privilégié.
- **[AssociateInstanceProfile](associate-instance-profile.md)** : attacher un profil d'instance privilégié à une instance que vous contrôlez déjà.

## Références

- [Rhino Security Labs: AWS privilege escalation (resource update paths)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
