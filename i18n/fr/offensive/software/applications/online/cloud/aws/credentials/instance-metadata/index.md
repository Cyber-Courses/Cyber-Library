---
order: 2
title: "Métadonnées d'instance"
description: "Voler des identifiants de rôle depuis le service de métadonnées d'instance EC2 et ECS : accès ouvert IMDSv1, flux de jeton IMDSv2, et atteindre 169.254.169.254 via un SSRF."
keywords:
  - IMDS
  - 169.254.169.254
  - rôle d'instance
  - IMDSv2
  - SSRF
---

# Métadonnées d'instance

Chaque instance EC2 et chaque tâche ECS dotée d'un rôle atteint ses identifiants via l'endpoint de métadonnées link-local à `169.254.169.254`. Tout ce qui peut émettre une requête HTTP depuis l'instance, un shell, une application web vulnérable ou un SSRF, peut demander au service de métadonnées les identifiants temporaires du rôle et repartir en tant que ce rôle.

## Ce qui s'y rattache

- **[IMDSv1](imdsv1.md)** : l'endpoint GET non authentifié, le cas le plus simple.
- **[IMDSv2](imdsv2.md)** : le flux jeton-d'abord et ce qu'il faut pour le satisfaire.
- **[SSRF](ssrf.md)** : atteindre l'endpoint via une server-side request forgery dans une application.

## L'endpoint

```bash
# role name, then its credentials (IMDSv1 form)
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>
# ECS task role uses a different path
curl http://169.254.170.2$AWS_CONTAINER_CREDENTIALS_RELATIVE_URI
```

## Références

- [AWS: instance metadata service (IMDSv2)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html)
- [HackTricks Cloud: AWS IMDS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
