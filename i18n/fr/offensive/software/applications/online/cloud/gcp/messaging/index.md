---
title: "Messagerie GCP"
order: 9
description: "Abuser de la messagerie GCP : accès aux sujets et abonnements Pub/Sub et aux files d'attente Cloud Tasks pour la capture et l'injection de données."
keywords:
  - Pub/Sub
  - Cloud Tasks
  - sujet
  - abonnement
  - file d'attente
---

# Messagerie

Les services de messagerie de GCP déplacent les données et déclenchent du travail, de sorte que leur accès signifie lire les événements d'autres systèmes ou injecter les vôtres. Un IAM étendu sur **Pub/Sub** vous permet de siphonner un sujet ou de publier des messages falsifiés, et les files d'attente **Cloud Tasks** peuvent mettre en file d'attente des requêtes qui appellent des points de terminaison en tant que compte de service choisi.

## Ce qui est regroupé ici

- **[Pub/Sub](pub-sub.md)** : lire ou publier sur des sujets et abonnements via un IAM étendu pour capturer ou injecter des messages.
- **[Cloud Tasks](cloud-tasks.md)** : mettre en file d'attente des requêtes qui appellent des points de terminaison ou des API en tant que compte de service choisi.

## Références

- [HackTricks Cloud : GCP Pub/Sub](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud : GCP](https://hackingthe.cloud/)
- [Google : contrôle d'accès Pub/Sub](https://cloud.google.com/pubsub/docs/access-control)
