---
title: "Hybrid Runbook Worker : exécuter sur des hôtes on-prem et des VM"
order: 3
description: "Exécuter des runbooks sur un Hybrid Runbook Worker pour exécuter du code sur des hôtes on-prem ou des VM."
keywords:
  - hybrid runbook worker
  - on-prem
  - code execution
  - runbook
  - lateral movement
---

# Hybrid Runbook Worker

Un **Hybrid Runbook Worker** exécute les runbooks Automation sur un serveur on-prem enregistré ou une VM Azure, au lieu du bac à sable Automation. Un runbook ciblant un groupe de workers hybrides s'exécute dans le contexte local du worker, généralement **SYSTEM**, ce qui transforme un accès en écriture au compte Automation en exécution de code sur ces hôtes et en passerelle du cloud vers le parc on-prem.

## Cibler un worker hybride

```bash
# list hybrid worker groups, then run a runbook on one
az automation hrwg list -g <rg> --automation-account-name <aa> -o table
az automation runbook start -g <rg> --automation-account-name <aa> \
  -n pwn --run-on <hybrid-worker-group>
```

```powershell
# runbook body: runs as SYSTEM on the worker host
whoami; hostname
# drop a payload / harvest local secrets on the on-prem box
```

## Notes d'exploitation

- Le runbook s'exécute avec les privilèges de l'agent worker, SYSTEM par défaut : il s'agit donc d'une exécution de code sur l'hôte, et pas seulement d'un jeton cloud.
- Les workers hybrides se trouvent souvent à l'intérieur du réseau de l'entreprise, ce qui en fait un pivot cloud-vers-on-prem ; poursuivez-le dans Active Directory.
- L'absence de `--run-on` fait exécuter le runbook dans le bac à sable Azure à la place ; le groupe hybride fait toute la différence entre un jeton cloud et un shell sur un serveur.

## Outils

- **Azure CLI** (`az automation hrwg`, `runbook start --run-on`).
- **MicroBurst** : énumération des workers hybrides et exécution de runbooks.

## Références

- [NetSPI: attacking Azure Automation hybrid workers](https://www.netspi.com/blog/)
- [HackTricks Cloud: Azure Automation](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Hybrid Runbook Worker overview](https://learn.microsoft.com/azure/automation/automation-hybrid-runbook-worker)
