---
title: "Automation Accounts"
order: 3
description: "Abusing Azure Automation Accounts: runbooks and the RunAs account for code execution as a privileged identity, plus hybrid worker reach."
keywords:
  - Automation Account
  - runbook
  - RunAs
  - hybrid worker
  - managed identity
---

# Automation Accounts

An Automation Account runs **runbooks** (PowerShell or Python) under its own identity, holds **credential, variable, and connection assets** that are often cleartext, and can reach on-prem or VM hosts through **Hybrid Runbook Workers**. It is one of the highest-value targets on the Azure resource plane: a runbook is arbitrary code as a privileged identity, and the account's assets routinely store service-principal secrets.

## What folds in here

- **[Runbooks](runbooks.md)**: writing and starting a runbook that runs as the account's identity.
- **[RunAs account](runas-account.md)**: the RunAs certificate and its service principal.
- **[Hybrid Runbook Worker](hybrid-runbook-worker.md)**: executing runbooks on on-prem or VM hosts.

The account's stored credential assets are harvested as [credentials](../../credentials/automation-assets.md).

## References

- [HackTricks Cloud: Azure Automation Account](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: running shells on Azure Automation hybrid workers](https://www.netspi.com/blog/)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
