---
title: "GCP messaging"
description: "Abusing GCP messaging: Pub/Sub topic and subscription access and Cloud Tasks queues for data capture and injection."
keywords:
  - Pub/Sub
  - Cloud Tasks
  - topic
  - subscription
  - queue
---

# Messaging

GCP's messaging services move data and trigger work, so access to them means reading other systems' events or injecting your own. Broad IAM on **Pub/Sub** lets you siphon a topic or publish forged messages, and **Cloud Tasks** queues can enqueue requests that call endpoints as a chosen service account.

## What folds in here

- **[Pub/Sub](pub-sub.md)**: reading or publishing to topics and subscriptions through broad IAM to capture or inject messages.
- **[Cloud Tasks](cloud-tasks.md)**: enqueuing requests that call endpoints or APIs as a chosen service account.

## References

- [HackTricks Cloud: GCP Pub/Sub](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: Pub/Sub access control](https://cloud.google.com/pubsub/docs/access-control)
