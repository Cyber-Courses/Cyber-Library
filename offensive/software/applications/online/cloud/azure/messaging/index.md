---
title: "Azure messaging"
description: "Abusing Azure messaging: Service Bus and Event Hubs streams, Event Grid routing, and Storage Queues for data capture and injection."
keywords:
  - Azure messaging
  - Service Bus
  - Event Hubs
  - Event Grid
  - Storage Queues
---

# Messaging

Azure's messaging services move application and telemetry data between producers and consumers, and access to them is usually a shared-access (SAS) key or a connection string rather than a scoped RBAC grant. When you hold that key, you read the data in flight, inject forged messages the consumer trusts, or reroute a stream to infrastructure you control.

## What folds in here

- **[Service Bus](service-bus.md)**: reading and injecting queue and topic messages through SAS or RBAC.
- **[Event Hubs](event-hubs.md)**: tapping a stream through a consumer group to capture telemetry and event data.
- **[Event Grid](event-grid.md)**: topic and subscription routing to intercept or inject events.
- **[Storage Queues](storage-queues.md)**: reading and writing queue messages through account keys or SAS.

## References

- [HackTricks Cloud: Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: Service Bus authentication and authorization](https://learn.microsoft.com/azure/service-bus-messaging/service-bus-authentication-and-authorization)
