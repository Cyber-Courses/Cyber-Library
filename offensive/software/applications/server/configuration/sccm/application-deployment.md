---
title: "SCCM application deployment: code as SYSTEM across the estate"
description: "Using Configuration Manager's own deployment features, once you hold the Full Administrator role or equivalent rights, to run applications, scripts, and CMPivot queries as SYSTEM on targeted device collections, turning SCCM into a built-in mass execution platform."
keywords:
  - SCCM application deployment
  - CMPivot
  - run scripts
  - device collection
  - SYSTEM execution
---

# SCCM application deployment

This is the payoff of [site takeover](site-takeover.md): SCCM exists to deliver software to clients, so a Full Administrator (or anyone with the right deployment rights) can point that machinery at chosen targets and execute **as SYSTEM** on them. No exploit is involved; you are using the product as designed, against the estate.

## Deploying to a collection

```bash
# SharpSCCM: deploy an application to a device or a collection, executed by the client agent as SYSTEM
SharpSCCM.exe exec -d <device> -p "C:\Windows\System32\cmd.exe /c <payload>"
SharpSCCM.exe exec -c <collection> -r <relay-or-command>
```

A **device collection** can be a single host or the whole estate, so the same action scales from one target to every managed machine.

## Run Scripts and CMPivot

Beyond full application deployments, the **Run Scripts** feature pushes PowerShell to clients, and **CMPivot** runs live queries across a collection. Both execute on the client and are faster and quieter than packaging an application:

```text
# CMPivot runs a live query across a collection (recon or execution primitive)
# Run Scripts pushes approved PowerShell to selected devices as SYSTEM
```

## Exploitation notes

- Deployment runs in the client agent's context, which is **SYSTEM**, so this is local-admin-equivalent on every target without touching their credentials.
- **SharpSCCM `exec`** can also force a client to authenticate to you (a coercion primitive) rather than run a payload, which pairs back into [relay](../../directory/active-directory/authentication/ntlm/relay.md).
- Collection scope is the blast radius: deploying to a broad collection is estate-wide code execution, so scope deliberately on an engagement.
- CMPivot is a useful **recon** tool even before full takeover if you hold read rights, inventorying every client live.

## Tools

- **SharpSCCM** (`exec`): application and command deployment, and client coercion.
- **Native console / Run Scripts / CMPivot**: deployment and live query once you hold administrative rights.

## References

- [SpecterOps: Misconfiguration Manager, EXEC](https://github.com/subat0mik/Misconfiguration-Manager/tree/main/attack-techniques/EXEC)
- [SharpSCCM (Mayyhem)](https://github.com/Mayyhem/SharpSCCM)
- [HTTP418InfoSec: offensive SCCM summary](https://http418infosec.com/offensive-sccm-summary)
