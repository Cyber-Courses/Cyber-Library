---
title: "Defender for Cloud: downgrading plans and policy to go dark"
description: "Disabling or downgrading Microsoft Defender for Cloud plans and policy to blind cloud threat detection."
keywords:
  - Defender for Cloud
  - security policy
  - alert suppression
  - detection evasion
---

# Defender for Cloud

Microsoft Defender for Cloud is the posture and threat-detection layer that raises alerts on suspicious resource behaviour (anomalous VM commands, Key Vault access, storage exfiltration). An operator with `Microsoft.Security/*` write drops its paid plans to **Free**, kills auto-provisioning of the agents that feed it, and suppresses the alerts it would raise.

## Downgrade the plans

```bash
# see which plans are on, then drop the ones watching your target to Free
az security pricing list -o table
az security pricing create -n VirtualMachines --tier Free
az security pricing create -n StorageAccounts --tier Free
az security pricing create -n KeyVaults --tier Free
```

## Kill telemetry and suppress alerts

```bash
# stop auto-provisioning the monitoring agent that feeds detections
az security auto-provisioning-setting update -n default --auto-provision Off

# create an alert suppression (dismiss) rule for the alert types you will trigger
az security alerts-suppression-rule update --rule-name quiet \
  --alert-type <AlertTypeName> --reason "Other" --state Enabled
```

## Exploitation notes

- Plan state is per-subscription; in a multi-subscription tenant, downgrade only where you operate to stay low-profile, since a tenant-wide change is itself conspicuous.
- Suppression rules are quieter than disabling the whole plan: the plan stays green while the specific alert you expect is auto-dismissed.
- These `Microsoft.Security` writes land in the [Activity Log](activity-log.md); time them against what still exports to a SIEM.
- Defender alerts often flow into [Sentinel](sentinel.md) through a connector, so disabling the connector there can blind the same alerts from the other end.

## Tools

- **Azure CLI** (`az security pricing`, `az security auto-provisioning-setting`, `az security alerts-suppression-rule`).
- **MicroBurst** / **PowerZure**: posture enumeration and bulk changes.

## References

- [HackTricks Cloud: Azure Defender for Cloud](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Stratus Red Team: Azure techniques](https://stratus-red-team.cloud/attack-techniques/azure/)
- [Microsoft: Defender for Cloud pricing API](https://learn.microsoft.com/rest/api/defenderforcloud/pricings)
