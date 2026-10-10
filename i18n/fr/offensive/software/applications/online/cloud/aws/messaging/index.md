---
title: "Messagerie AWS"
order: 9
description: "Abuser de la messagerie AWS : SES pour l'envoi de mail et le phishing, et l'accès aux topics et files SNS et SQS pour la capture de données et l'injection de messages."
keywords:
  - SES
  - SNS
  - SQS
  - phishing
  - message queue
  - notification
  - pub/sub
---

# Messagerie

Les services de messagerie AWS transportent le mail et les événements applicatifs, et chacun est utile à un attaquant détenant la bonne permission. **SES** envoie du mail au nom de la victime, en empruntant la réputation d'expédition établie d'un domaine pour du phishing. **SNS** et **SQS** déplacent des messages applicatifs : l'accès aux topics et aux files signifie donc lire les données qui y transitent, injecter des événements falsifiés ou supprimer des messages dont un système dépend.

## Ce qui est regroupé ici

- **[SES](ses.md)** : envoyer du phishing et du mail usurpé depuis des identités vérifiées, et lire le quota d'envoi du compte et les domaines vérifiés.
- **[SNS](sns.md)** : publier sur des topics, s'abonner pour capturer des notifications, et exfiltrer via un abonnement.
- **[SQS](sqs.md)** : lire, injecter et supprimer des messages dans une file exposée par sa politique de ressource.

## Références

- [HackTricks Cloud : AWS SES, SNS, SQS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS : autorisation d'envoi Amazon SES](https://docs.aws.amazon.com/ses/latest/dg/sending-authorization.html)
- [Rhino Security Labs : recherche offensive AWS](https://rhinosecuritylabs.com/aws/)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
