---
title: "Activity Log: evading and starving the subscription operation record"
description: "Evading or clearing the Azure Activity Log and subscription-level operation records."
keywords:
  - Activity Log
  - operation log
  - subscription
  - detection evasion
  - anti-forensics
---

# Activity Log

The Activity Log is the subscription-level record of control-plane operations (who called which ARM write, on what resource). It cannot be deleted or edited and is held for 90 days, so it is not blinded directly. It is blinded indirectly: by cutting the **diagnostic settings** that copy it into a Log Analytics workspace, Event Hub, or storage account where a SIEM and long-term retention actually live, and by preferring operations the log never captures.

## See what it captures first

```bash
# recent control-plane operations (recon your own noise before acting)
az monitor activity-log list --offset 2h --query "[].{op:operationName.value,caller:caller,status:status.value}" -o table
```

## Cut the export to downstream sinks

The Activity Log stays, but stop it flowing to the subscription's log pipeline:

```bash
# list subscription-level Activity Log exports, then remove them
az monitor diagnostic-settings subscription list
az monitor diagnostic-settings subscription delete --name <setting>
```

## Exploitation notes

- The log records ARM control-plane writes only. **Data-plane** actions (reading a blob, pulling a Key Vault secret over its data endpoint, querying a database) do not appear here, so data-plane tradecraft is quiet by construction; see the data-plane paths under [credentials](../credentials/index.md) and [storage](../storage/index.md).
- `DELETE` on a subscription diagnostic setting is itself an Activity Log event, so it lands in the 90-day store; it only severs the downstream SIEM copy, it does not erase the native record.
- Alerting typically rides on the exported copy in Log Analytics, so cutting the export is what actually stops rules from firing; pair with the [Azure Monitor](azure-monitor.md) diagnostic-setting teardown at resource scope.

## Tools

- **Azure CLI** (`az monitor activity-log`, `az monitor diagnostic-settings subscription`).
- **MicroBurst**: subscription enumeration to map where logs are shipped.

## References

- [HackTricks Cloud: Azure monitoring and logging](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Stratus Red Team: Azure techniques](https://stratus-red-team.cloud/attack-techniques/azure/)
- [Microsoft: Azure Activity Log](https://learn.microsoft.com/azure/azure-monitor/essentials/activity-log)
