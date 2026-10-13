---
order: 4
title: "AMI : secrets dans les images machine partagées et publiques"
description: "Trouver des secrets dans des AMI partagées ou publiques et lancer des instances à partir d'elles pour hériter d'un accès préétabli."
keywords:
  - AMI
  - image machine
  - public
  - secrets
  - lancement
---

# AMI

Une **Amazon Machine Image** est un volume racine packagé avec des métadonnées de lancement. Les équipes intègrent des identifiants, des clés et des secrets applicatifs dans des AMI de référence (golden AMIs), puis les partagent trop largement, si bien qu'une AMI partagée ou publique est à la fois une fuite de données et un moyen de lancer un hôte qui fait déjà confiance à un accès préparé par l'attaquant.

## Trouver et utiliser des AMI

```bash
# AMIs an account owns, and ones shared with you
aws ec2 describe-images --owners <acct>
aws ec2 describe-images --executable-users self

# Launch from a shared AMI, or make a volume from its snapshot to read offline
aws ec2 run-instances --image-id ami-xxxx --instance-type t3.micro --key-name <k>
```

Une AMI référence des snapshots EBS ; monter ces snapshots (voir [snapshots](snapshots.md)) permet de lire le système de fichiers intégré sans démarrer l'image.

## Notes d'exploitation

- Les AMI publiques (`--executable-users all`) sont énumérables à l'échelle d'une région et portent parfois encore des secrets vivants issus d'une publication par erreur.
- Lancer l'image fait hériter de toutes les clés SSH, tâches cron ou agents intégrés, si bien qu'une image de référence empoisonnée ou obsolète peut vous offrir une persistance.
- À combiner avec [Image Builder](../image-builder.md) : empoisonner le pipeline qui produit une AMI étend la compromission à chaque hôte lancé à partir d'elle.

## Outils

- **AWS CLI** (`describe-images`, `run-instances`) : découverte et lancement.
- **dsnap** : lire le snapshot sous-jacent de l'AMI hors ligne.

## Références

- [HackTricks Cloud: AMIs](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [AWS: describe-images](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-images.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
