---
title: "Token impersonation: SeImpersonate to SYSTEM"
order: 14
description: "Turning the SeImpersonatePrivilege that service accounts hold into SYSTEM by coercing a privileged process to authenticate to a controlled endpoint and stealing its token, the potato family of techniques that bridges a service-account foothold to full local control."
keywords:
  - SeImpersonatePrivilege
  - token impersonation
  - PrintSpoofer
  - RoguePotato
  - GodPotato
---

# Token impersonation

When you land on a host as a **service account** (IIS, MSSQL, and most service identities), you usually hold **`SeImpersonatePrivilege`**: the right to impersonate a client that authenticates to you. That is the whole game. Coerce a **SYSTEM** process to authenticate to an endpoint you control, capture and impersonate its token, and you run as SYSTEM. This "potato" family is the standard bridge from a constrained service-account foothold to full local control, and from there to [LSASS](lsass-dumping.md) and the rest of the host's secrets.

## Confirm the privilege

```cmd
whoami /priv
:: look for SeImpersonatePrivilege or SeAssignPrimaryTokenPrivilege = Enabled
```

## The variants

They differ only in **how they coerce a SYSTEM process to authenticate**:

```cmd
:: PrintSpoofer: spooler RpcRemoteFindFirstPrinterChangeNotification over a named pipe (modern default)
PrintSpoofer.exe -i -c cmd.exe

:: GodPotato: a broad DCOM/RPC method that works across modern Windows versions
GodPotato.exe -cmd "cmd /c whoami"

:: RoguePotato: DCOM OXID resolver abuse for post-1809 systems
:: JuicyPotato: the original DCOM method, for legacy targets (pre-1809)
```

## A lineage of potatoes

The family has tracked Microsoft's attempts to close each coercion channel:

- **2016, Hot/RottenPotato**: the original NBNS + DCOM token-capture idea.
- **2018, JuicyPotato**: weaponised the DCOM path broadly, until the 1809 OXID change broke it.
- **2019-2020, RoguePotato and PrintSpoofer**: restored DCOM abuse post-1809 and added the spooler named-pipe path.
- **2022 onward, GodPotato and the DCOM/RPC successors**: broad methods that keep working on current Windows.

## Exploitation notes

- The privilege is held by **default** by service accounts, so a web shell or SQL `xp_cmdshell` as the service identity is almost always one step from SYSTEM.
- **PrintSpoofer** is the usual first try on modern hosts; fall back to **GodPotato**/RoguePotato for the DCOM path and JuicyPotato on legacy.
- SYSTEM on a domain-joined host means its **machine account** and any cached credentials, so this pivots straight into domain movement via [pass-the-hash](../ntlm/pass-the-hash.md) or [coercion](../ntlm/coercion.md).
- These abuse host-local token APIs, not AD, so they work regardless of domain hardening once you hold a service-account context.

## Tools

- **PrintSpoofer** (itm4n): spooler named-pipe coercion.
- **GodPotato** / **RoguePotato** (antonioCoco): modern and post-1809 DCOM coercion.
- **JuicyPotato**: the legacy DCOM technique.

## References

- [itm4n: PrintSpoofer, abusing impersonation privileges](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/)
- [GodPotato (antonioCoco)](https://github.com/BeichenDream/GodPotato)
- [HackTricks: abusing tokens and SeImpersonate](https://hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/privilege-escalation-abusing-tokens.html)
