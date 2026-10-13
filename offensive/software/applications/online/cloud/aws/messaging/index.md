---
title: "AWS messaging"
order: 9
description: "Abusing AWS messaging: SES for mail sending and phishing, and SNS and SQS topic and queue access for data capture and message injection."
keywords:
  - SES
  - SNS
  - SQS
  - phishing
  - message queue
  - notification
  - pub/sub
---

# Messaging

AWS messaging services carry mail and application events, and each one is useful to an attacker holding the right permission. **SES** sends mail as the victim, borrowing a domain's established sending reputation for phishing. **SNS** and **SQS** move application messages, so topic and queue access means reading the data flowing through, injecting forged events, or dropping messages a system depends on.

## What folds in here

- **[SES](ses.md)**: sending phishing and spoofed mail from verified identities, and reading the account's sending quota and verified domains.
- **[SNS](sns.md)**: publishing to topics, subscribing to capture notifications, and exfiltrating through a subscription.
- **[SQS](sqs.md)**: reading, injecting, and deleting messages in a queue exposed by its resource policy.

## References

- [HackTricks Cloud: AWS SES, SNS, SQS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: Amazon SES sending authorization](https://docs.aws.amazon.com/ses/latest/dg/sending-authorization.html)
- [Rhino Security Labs: AWS offensive research](https://rhinosecuritylabs.com/aws/)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
