---
title: "Session Manager : shells interactifs via ssm:StartSession"
description: "ssm:StartSession pour ouvrir un shell interactif sur une instance managée sous les privilèges de l'agent SSM."
keywords:
  - Session Manager
  - StartSession
  - shell
  - agent SSM
  - interactif
---

# Session Manager

`ssm:StartSession` ouvre un shell interactif sur une instance managée, tunnelé à travers le service SSM plutôt que par un port réseau. C'est le pendant interactif de run command : les mêmes privilèges d'agent (root ou SYSTEM), la même propriété d'absence d'accès entrant, mais un terminal en direct pour le travail pratique.

## Ouvrir une session

```bash
# needs the session-manager-plugin installed locally
aws ssm start-session --target <instance-id>

# port forwarding to reach an internal service through the instance
aws ssm start-session --target <id> \
  --document-name AWS-StartPortForwardingSession \
  --parameters '{"portNumber":["3306"],"localPortNumber":["3306"]}'
```

## Notes d'exploitation

- Le document de redirection de port transforme une instance managée en pivot vers les sous-réseaux privés sans ouvrir de security group.
- Les sessions sont journalisées dans CloudTrail et, en option, dans S3 ou CloudWatch ; le chemin run command peut être plus discret pour une action unique.
- Comme avec run command, le shell s'exécute en tant qu'agent et atteint le rôle d'instance via IMDS.

## Outils

- **AWS CLI** + **session-manager-plugin** : démarrer des sessions et des redirections de port.

## Références

- [AWS: Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
- [HackTricks Cloud: Session Manager](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
