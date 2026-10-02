---
title: "Editing a GPO: code execution across its scope"
description: "Turning write access over an Active Directory Group Policy Object into code execution on every computer or user it applies to, by injecting an immediate scheduled task, startup or logon script, or a local-administrator membership, with pyGPOAbuse and SharpGPOAbuse."
keywords:
  - GPO abuse
  - immediate scheduled task
  - pyGPOAbuse
  - SharpGPOAbuse
  - SYSVOL
---

# Editing a GPO

A GPO you can **write at the directory object** (a `GenericWrite`/`GenericAll`/`WriteProperty` edge) is code execution on everything the GPO is linked to. You add a policy item, the client machines fetch it on their next refresh, and it runs. The highest-impact item is an **immediate scheduled task**, which runs once as soon as the policy applies, as **SYSTEM** on computers in scope.

Injecting a **new** item takes more than the SYSVOL files: a client only processes a policy area whose client-side extension is registered in the GPO object's **`gPCMachineExtensionNames`** (or `gPCUserExtensionNames`), and only when the object's **`versionNumber`** increments. Those are LDAP attributes on the GPO object, so a brand-new scheduled task needs the directory-object write, not just the SYSVOL folder. Write access to **only** the SYSVOL folder is weaker: it lets you tamper with content the GPO **already** references (an existing script or task), not add a new policy area. The tools below perform both the SYSVOL write and the directory updates, so they assume object write.

## What you can push

- **Immediate scheduled task**: the fastest and most reliable; runs your command as SYSTEM (computer GPO) or as the user (user GPO) at the next refresh.
- **Startup / logon script**: runs at boot (SYSTEM) or logon (user), persistent across refreshes until removed.
- **Local administrators membership**: add a controlled account to the local Administrators group of every in-scope machine (a Restricted Groups / LGPO change).
- **Services, registry autoruns**: alternative execution and persistence through policy-delivered configuration.

## Doing it from Linux and Windows

```bash
# pyGPOAbuse (Linux): add an immediate scheduled task to a writable GPO.
# Default action makes a local admin; -command runs an arbitrary payload as SYSTEM.
pygpoabuse.py example.local/user -hashes :<nthash> -gpo-id '<GPO-GUID>' \
  -command 'net user hacker P@ss123! /add && net localgroup administrators hacker /add' \
  -taskname 'Update' -description 'Update task'
```

```powershell
# SharpGPOAbuse (Windows): local admin, or an arbitrary computer task
SharpGPOAbuse.exe --AddLocalAdmin --UserAccount svc_backup --GPOName "Vulnerable GPO"
SharpGPOAbuse.exe --AddComputerTask --TaskName "Update" --Author DOMAIN\user `
  --Command "cmd.exe" --Arguments "/c <payload>" --GPOName "Vulnerable GPO"

# PowerView: immediate task via the GPO display name
New-GPOImmediateTask -TaskName Update -Command cmd -CommandArguments "/c <payload>" -GPODisplayName "Vulnerable GPO" -Force
```

These tools write the policy file (for example `ScheduledTasks.xml`) into the GPO's SYSVOL folder and bump the GPO version (`GPT.ini`) and the client-side-extension list (`gPCMachineExtensionNames`) so clients process it.

## Exploitation notes

- Application is on the **Group Policy refresh cycle** (~90 minutes plus jitter on workstations, 5 minutes on DCs), or immediately with `gpupdate /force` if you can run it; an immediate task then fires on the next apply.
- **Scope is everything**: a GPO linked to the Domain Controllers OU runs your task as SYSTEM on the DCs, which is domain compromise. Confirm the GPO's links (see [enumeration](gpo-and-ou-enumeration.md)) before firing.
- **Clean up**: remove the task/script and restore the version number afterwards; a lingering immediate task re-triggers and is an obvious artifact in SYSVOL.
- A computer-scoped immediate task runs as SYSTEM and needs no user to log on, so it is preferred over logon scripts for reliability.

## Tools

- **pyGPOAbuse**: immediate scheduled task / local admin from Linux.
- **SharpGPOAbuse**: `--AddLocalAdmin`, `--AddComputerTask`, `--AddUserTask`, `--AddUserRights` on Windows.
- **PowerView `New-GPOImmediateTask`**: immediate task from a shell.

## References

- The Hacker Recipes: Group policies
- SpecterOps: GPO abuse and the GenericWrite-over-GPO edge
