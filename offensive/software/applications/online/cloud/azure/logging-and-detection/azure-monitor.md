---
title: "Azure Monitor: killing diagnostic settings and stealing workspace keys"
description: "Tampering with Azure Monitor and Log Analytics: disabling diagnostic settings and stealing workspace shared keys."
keywords:
  - Azure Monitor
  - Log Analytics
  - diagnostic settings
  - workspace key
  - log tampering
---

# Azure Monitor

Azure Monitor is the pipe that carries resource logs and metrics into a **Log Analytics** workspace, where Sentinel and alert rules read them. Two offensive moves matter: delete the **diagnostic settings** on a resource so its logs stop flowing, and steal the workspace **shared keys** so you can forge or flood ingestion.

## Stop the logs at the resource

```bash
# find what a resource ships and where, then remove it
az monitor diagnostic-settings list --resource <resource-id> -o table
az monitor diagnostic-settings delete --name <setting> --resource <resource-id>
```

Without a diagnostic setting the resource keeps running but its logs never reach the workspace, so nothing downstream can alert on them.

## Steal the workspace keys

```bash
# primary/secondary shared keys for the Log Analytics workspace
az monitor log-analytics workspace get-shared-keys -g <rg> -n <workspace>
```

The shared key authorizes ingestion through the HTTP Data Collector API: with it you can inject fabricated records to bury real events in noise, or spoof benign activity.

## Exploitation notes

- Diagnostic settings are per-resource, so blinding one VM or Key Vault does not touch the rest; enumerate the whole set before relying on silence.
- Data Collection Rules (DCRs) feed the newer Azure Monitor Agent pipeline; delete or detach the DCR (`az monitor data-collection rule`) as well as classic diagnostic settings.
- The workspace key is a long-lived secret; it is also a persistence foothold for continued log injection after other access is lost.
- Deleting a setting is an Activity Log event (see [Activity Log](activity-log.md)); it severs the SIEM feed but is itself recorded.

## Tools

- **Azure CLI** (`az monitor diagnostic-settings`, `az monitor log-analytics workspace get-shared-keys`, `az monitor data-collection rule`).
- **MicroBurst**: workspace and key enumeration across a subscription.

## References

- [HackTricks Cloud: Azure monitoring](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [MicroBurst (NetSPI)](https://github.com/NetSPI/MicroBurst)
- [Microsoft: Log Analytics data collector API](https://learn.microsoft.com/azure/azure-monitor/logs/data-collector-api)
