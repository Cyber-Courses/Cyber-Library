---
order: 2
title: "SSM"
description: "Exécuter des commandes sur des instances managées via Systems Manager : ssm:SendCommand et shells Session Manager sous le rôle de l'instance."
keywords:
  - SSM
  - SendCommand
  - Session Manager
  - instance managée
  - exécution de commande
---

# SSM

AWS Systems Manager (SSM) est un canal légitime d'administration à distance, et c'est l'un des moyens les plus propres d'exécuter du code sur EC2 depuis la seule API. Toute instance exécutant l'agent SSM avec un rôle qui l'autorise est atteignable : pas de clé SSH, pas de port ouvert, pas besoin de toucher au service de métadonnées. L'exécution de commande se fait en tant que root ou SYSTEM, ce qui livre ensuite le rôle d'instance et les secrets présents sur la machine.

## Ce qui s'y rattache

- **[Run command](run-command.md)** : `ssm:SendCommand` pour exécuter des commandes sur une ou plusieurs instances managées.
- **[Session Manager](session-manager.md)** : `ssm:StartSession` pour ouvrir un shell interactif.

## Trouver les instances managées

```bash
aws ssm describe-instance-information --query 'InstanceInformationList[].InstanceId'
```

## Références

- [HackTricks Cloud: SSM abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [AWS: Systems Manager Run Command](https://docs.aws.amazon.com/systems-manager/latest/userguide/execute-remote-commands.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
