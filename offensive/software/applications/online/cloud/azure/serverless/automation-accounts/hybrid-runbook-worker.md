---
title: "Hybrid Runbook Worker: executing on on-prem and VM hosts"
order: 3
description: "Executing runbooks on a Hybrid Runbook Worker to run code on on-prem or VM hosts."
keywords:
  - hybrid runbook worker
  - on-prem
  - code execution
  - runbook
  - lateral movement
---

# Hybrid Runbook Worker

A **Hybrid Runbook Worker** runs Automation runbooks on a registered on-prem server or Azure VM instead of the Automation sandbox. A runbook targeted at a hybrid worker group executes as the worker's local context, usually **SYSTEM**, which turns Automation Account write access into code execution on those hosts and a bridge from cloud into the on-prem estate.

## Targeting a hybrid worker

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

## Exploitation notes

- The runbook runs with the worker agent's privileges, SYSTEM by default, so this is host code execution, not just a cloud token.
- Hybrid workers commonly sit inside the corporate network, making this a cloud-to-on-prem pivot; follow it into Active Directory.
- No `--run-on` means the runbook executes in the Azure sandbox instead; the hybrid group is the difference between a cloud token and a shell on a server.

## Tools

- **Azure CLI** (`az automation hrwg`, `runbook start --run-on`).
- **MicroBurst**: hybrid worker enumeration and runbook execution.

## References

- [NetSPI: attacking Azure Automation hybrid workers](https://www.netspi.com/blog/)
- [HackTricks Cloud: Azure Automation](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Hybrid Runbook Worker overview](https://learn.microsoft.com/azure/automation/automation-hybrid-runbook-worker)
