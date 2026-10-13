---
title: "Automation Accounts"
order: 3
description: "Détourner les Azure Automation Accounts : runbooks et compte RunAs pour l'exécution de code en tant qu'identité privilégiée, et portée des workers hybrides."
keywords:
  - Automation Account
  - runbook
  - RunAs
  - hybrid worker
  - managed identity
---

# Automation Accounts

Un Automation Account exécute des **runbooks** (PowerShell ou Python) sous sa propre identité, détient des **assets de type credential, variable et connection** souvent en clair, et peut atteindre des hôtes on-prem ou des VM via des **Hybrid Runbook Workers**. C'est l'une des cibles les plus précieuses du plan des ressources Azure : un runbook est du code arbitraire exécuté en tant qu'identité privilégiée, et les assets du compte stockent couramment des secrets de service principal.

## Ce qui se rattache ici

- **[Runbooks](runbooks.md)** : écrire et démarrer un runbook qui s'exécute en tant qu'identité du compte.
- **[Compte RunAs](runas-account.md)** : le certificat RunAs et son service principal.
- **[Hybrid Runbook Worker](hybrid-runbook-worker.md)** : exécuter des runbooks sur des hôtes on-prem ou des VM.

Les assets de type credential stockés par le compte sont récoltés en tant que [credentials](../../credentials/automation-assets.md).

## Références

- [HackTricks Cloud: Azure Automation Account](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: running shells on Azure Automation hybrid workers](https://www.netspi.com/blog/)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
