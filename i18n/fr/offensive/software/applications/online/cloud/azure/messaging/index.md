---
title: "Messagerie Azure"
order: 9
description: "Abuser de la messagerie Azure : flux Service Bus et Event Hubs, routage Event Grid et Storage Queues pour la capture et l'injection de données."
keywords:
  - messagerie Azure
  - Service Bus
  - Event Hubs
  - Event Grid
  - Storage Queues
---

# Messagerie

Les services de messagerie d'Azure déplacent les données applicatives et de télémétrie entre producteurs et consommateurs, et leur accès repose généralement sur une clé d'accès partagé (SAS) ou une connection string plutôt que sur un octroi RBAC restreint. Lorsque vous détenez cette clé, vous lisez les données en transit, injectez des messages forgés sur lesquels le consommateur se fie, ou reroutez un flux vers une infrastructure que vous contrôlez.

## Ce qui se rattache ici

- **[Service Bus](service-bus.md)** : lire et injecter des messages de queue et de topic via SAS ou RBAC.
- **[Event Hubs](event-hubs.md)** : écouter un flux via un consumer group pour capturer la télémétrie et les données d'événements.
- **[Event Grid](event-grid.md)** : routage de topics et de subscriptions pour intercepter ou injecter des événements.
- **[Storage Queues](storage-queues.md)** : lire et écrire des messages de queue via des clés de compte ou SAS.

## Références

- [HackTricks Cloud : Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft : authentification et autorisation Service Bus](https://learn.microsoft.com/azure/service-bus-messaging/service-bus-authentication-and-authorization)
