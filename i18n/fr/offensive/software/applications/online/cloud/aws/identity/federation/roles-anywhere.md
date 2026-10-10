---
title: "Roles Anywhere : obtenir des identifiants de rôle depuis un certificat d'ancre de confiance"
order: 5
description: "Abuser des ancres de confiance IAM Roles Anywhere et des certificats client pour obtenir des identifiants de rôle depuis l'extérieur d'AWS."
keywords:
  - Roles Anywhere
  - ancre de confiance
  - certificat
  - X.509
  - identifiants de rôle
---

# Roles Anywhere

IAM Roles Anywhere permet à des charges de travail situées hors d'AWS d'obtenir des identifiants de rôle en présentant un certificat client X.509 émis sous une **ancre de confiance** configurée (une AC). Si vous parvenez à obtenir ou à émettre un certificat accepté par l'ancre de confiance, vous générez des identifiants de rôle depuis n'importe où, sans aucun utilisateur AWS.

## Échanger un certificat contre des identifiants

```bash
aws_signing_helper credential-process \
  --certificate client.pem --private-key client.key \
  --trust-anchor-arn arn:aws:rolesanywhere:...:trust-anchor/<id> \
  --profile-arn arn:aws:rolesanywhere:...:profile/<id> \
  --role-arn arn:aws:iam::<acct>:role/<role>
```

## Notes d'exploitation

- L'attaque repose entièrement sur l'AC : le contrôle de l'AC de l'ancre de confiance (ou d'une sous-AC qui s'y rattache) vous permet d'émettre des certificats pour tout rôle autorisé par le profil.
- Un certificat client et sa clé divulgués sont directement réutilisables jusqu'à leur révocation ou leur expiration.
- Le profil et le rôle limitent quels rôles un certificat peut endosser : énumérez donc les profils pour voir l'ensemble des cibles accessibles.

## Outils

- **aws_signing_helper** : l'assistant d'identifiants AWS pour Roles Anywhere.
- Outillage PKI standard (OpenSSL) pour inspecter ou émettre des certificats.

## Références

- [AWS: IAM Roles Anywhere](https://docs.aws.amazon.com/rolesanywhere/latest/userguide/introduction.html)
- [HackTricks Cloud: AWS Roles Anywhere](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
