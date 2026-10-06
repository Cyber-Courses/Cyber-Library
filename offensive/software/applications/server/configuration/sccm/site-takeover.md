---
title: "SCCM site takeover: relaying to Full Administrator"
description: "Taking control of a Configuration Manager site by coercing the site server's authentication and relaying it to the site database (MSSQL) or the SMS Provider, granting the Full Administrator role and with it SYSTEM execution on every managed device."
keywords:
  - SCCM site takeover
  - NTLM relay
  - MSSQL site database
  - SMS Provider
  - Full Administrator
---

# Site takeover

Taking the **Full Administrator** role in a site is the top prize: it is remote code execution as SYSTEM on **every device in the site**. The modern path does not need an SCCM credential at all. It coerces the **site server's** machine account into authenticating and [relays](../../directory/active-directory/authentication/ntlm/relay.md) that authentication to a component that trusts it, either the site database or the SMS Provider.

## Relay to the site database (MSSQL)

The site server's machine account is a sysadmin on its own **site database**. Coerce it, relay to MSSQL, and write yourself a Full Administrator record:

```bash
# Relay the coerced site-server auth to the site database and add an SCCM admin
SharpSCCM.exe ... # enumerate the site database target first
ntlmrelayx.py -t mssql://<site-db> -q "<SCCM admin insert>" -smb2support
# coerce the site server (PetitPotam / Coercer) so its machine account authenticates to the relay
```

## Relay to the SMS Provider

The **SMS Provider** is the WMI/administration layer; relaying a privileged authentication to it over SMB or HTTP can likewise register a new Full Administrator, and is the route when the database is not directly reachable.

## Hierarchy takeover

Where a **central administration site** sits above primary sites, taking the top of the hierarchy cascades Full Administrator down to every child primary, so note the hierarchy during [recon](reconnaissance.md) and target the highest reachable site.

## Exploitation notes

- The coercion targets the **site server's machine account**, so this needs no SCCM role and often no more than network access plus a coercion primitive.
- It chains directly with the AD relay toolchain: the same [coercion](../../directory/active-directory/authentication/ntlm/coercion.md) and `ntlmrelayx` you use against a DC apply here, only the relay target changes.
- Full Administrator is **SYSTEM on every client** via [application deployment](application-deployment.md), so site takeover is effectively estate-wide compromise, not just SCCM control.
- Where relay is blocked, fall back to the [credential](credential-harvesting.md) paths or to relaying the site server to [AD CS](../../directory/active-directory/authentication/certificates/index.md) instead.

## Tools

- **SharpSCCM / sccmhunter**: identify the site database and SMS Provider, and add administrators post-relay.
- **ntlmrelayx.py**: relay the coerced authentication to MSSQL or the SMS Provider.
- **Coercer / PetitPotam**: force the site server's machine account to authenticate.

## References

- [SpecterOps: Misconfiguration Manager, TAKEOVER](https://github.com/subat0mik/Misconfiguration-Manager/tree/main/attack-techniques/TAKEOVER)
- [Truesec: SCCM tier killer](https://www.truesec.com/hub/blog/sccm-tier-killer)
- [logan-goins: attacking and defending Configuration Manager](http://logan-goins.com/2025-04-25-sccm/)
