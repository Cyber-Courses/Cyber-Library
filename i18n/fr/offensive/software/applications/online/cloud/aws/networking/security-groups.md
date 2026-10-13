---
title: "Security groups : repérer les ports et services exposés sur Internet"
order: 1
description: "Cartographier et détourner les security groups trop permissifs qui exposent des ports de management et des services internes sur Internet."
keywords:
  - security group
  - ingress
  - 0.0.0.0/0
  - exposure
  - port
---

# Security groups

Un security group est une liste d'autorisation stateful de règles d'ingress et d'egress liée à une ENI. L'erreur courante est une règle d'ingress avec une source `0.0.0.0/0` sur un port de management ou de base de données, qui place SSH, RDP ou une base de données directement sur Internet. Avec `ec2:Describe*`, vous lisez chaque groupe et déterminez quelles instances il met en frontal, transformant la posture réseau du compte en liste de cibles.

## Repérer un ingress grand ouvert

```bash
# every rule that allows the whole internet
aws ec2 describe-security-groups \
  --query "SecurityGroups[?IpPermissions[?contains(IpRanges[].CidrIp, '0.0.0.0/0')]].{Id:GroupId,Name:GroupName}"

# which ports are exposed on each group
aws ec2 describe-security-groups \
  --query "SecurityGroups[].IpPermissions[?contains(IpRanges[].CidrIp,'0.0.0.0/0')].[FromPort,ToPort]"
```

## Associer les groupes aux hôtes actifs

```bash
# instances behind a group, with their public addresses
aws ec2 describe-instances \
  --filters Name=instance.group-id,Values=<sg-id> \
  --query "Reservations[].Instances[].[InstanceId,PublicIpAddress,PrivateIpAddress]"
```

## Notes d'exploitation

- `22`, `3389`, `5432`, `3306`, `6379`, `27017` et `9200` ouverts sur `0.0.0.0/0` sont les trouvailles à forte valeur : ils mettent en frontal SSH, RDP et des data stores non authentifiés ou faiblement authentifiés.
- Les règles à groupe référencé (la source est un autre security group, pas un CIDR) ne comptent qu'une fois que vous êtes à l'intérieur du VPC, alors associez cela au pivotement par [VPC](vpc.md).
- Les règles d'egress comptent pour votre propre chemin d'exfiltration depuis une instance compromise ; un groupe qui autorise tout l'egress est une porte ouverte vers l'extérieur.

## Outils

- **AWS CLI** (`ec2 describe-security-groups`, `describe-instances`) : l'énumération.
- **ScoutSuite** / **Prowler** : signaler les ports exposés sur Internet à l'échelle du compte.
- **nmap** / **masscan** : confirmer l'accessibilité des ports exposés depuis l'extérieur.

## Références

- [HackTricks Cloud : AWS EC2 et VPC](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-vpc-and-network-security.html)
- [AWS : security groups pour votre VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
- [CloudFox : trouver les ressources exposées sur Internet et les chemins réseau](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
