---
title: "Print Spooler: coercion and driver-loading code execution"
description: "The Windows Print Spooler as an attack surface: the PrinterBug coercion primitive that forces a machine to authenticate to an attacker over MS-RPRN, and the Point and Print driver-loading flaws (PrintNightmare, SpoolFool) that run code as SYSTEM."
keywords:
  - print spooler
  - PrinterBug
  - PrintNightmare
  - MS-RPRN
  - Point and Print
---

# Print Spooler

The Windows **Print Spooler** runs by default on workstations and servers, including domain controllers, and exposes remote RPC interfaces (MS-RPRN, MS-PAR) to any authenticated user. It has been one of the most reliably abused services in Windows for years, in two distinct ways: it can be made to **authenticate to an attacker** (coercion), and it can be made to **load an attacker's driver DLL as SYSTEM** (code execution).

## A long history of abuse

The spooler's offensive value has compounded over several waves, and each is still useful where patching or hardening lagged:

- **2018, PrinterBug / SpoolSample**: Lee Christensen showed that `RpcRemoteFindFirstPrinterChangeNotificationEx` forces a target to authenticate back over SMB, a dependable [coercion](coercion.md) primitive still used to feed relay and delegation attacks.
- **2021, PrintNightmare**: a flaw in `RpcAddPrinterDriverEx` and Point and Print let an unprivileged user load a driver DLL and execute as SYSTEM, both remotely and locally, covered under [Point and Print](point-and-print.md).
- **2022, SpoolFool** and later spooler privilege-escalation bugs continued the pattern of local SYSTEM through the same service.

## Pages

- **[Coercion](coercion.md)**: forcing a machine to authenticate with the PrinterBug, to feed relay and delegation.
- **[Point and Print](point-and-print.md)**: loading a driver to execute as SYSTEM, remotely and locally.

## References

- [HackTricks: print spooler service abuse](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/printers-spooler-service-abuse.html)
- [itm4n: a practical guide to PrintNightmare in 2024](https://itm4n.github.io/printnightmare-exploitation/)
- [SpecterOps: SpoolSample / PrinterBug origin](https://github.com/leechristensen/SpoolSample)
