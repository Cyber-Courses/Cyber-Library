---
title: "Out-of-band SQL injection in MySQL"
description: "Exfiltrating MySQL data over the network with LOAD_FILE and UNC paths when no in-band channel exists, and why MySQL out-of-band is largely Windows-only."
keywords:
  - out of band injection
  - OOB exfiltration
  - LOAD_FILE UNC
  - DNS exfiltration
  - NTLM hash leak
---

# Out-of-band

Out-of-band (OOB) injection sends data to a server the attacker controls instead of reading it from the response. It is the option when the result is not reflected, errors are suppressed, and timing is too slow or unreliable, provided the database can be made to reach the network.

MySQL's OOB surface is narrow and mostly Windows. It has no built-in HTTP or DNS resolver functions like Oracle's `UTL_HTTP` or `UTL_INADDR`, so the only general trigger is `LOAD_FILE()` pointed at a UNC path (`\\host\share\...`). On Windows, resolving and connecting to that path makes the host perform a DNS lookup and an SMB connection to the attacker, which both exfiltrates data encoded in the hostname and leaks the service account's NetNTLM hash. On Linux builds `LOAD_FILE` does not follow UNC paths, so this channel generally does not apply.

Every OOB path here needs the `FILE` privilege and a `secure_file_priv` setting that does not block the operation, the same gate as file read and write.

## Pages

- **[DNS exfiltration](dns-exfiltration.md)**: encode data into a UNC hostname so a DNS lookup leaks it.
- **[UNC and NTLM capture](unc-ntlm.md)**: force SMB authentication to capture the service account hash.

## Tools

- **sqlmap**: automates out-of-band DNS exfiltration with `--dns-domain`.
- **Burp Collaborator**: logging name server for DNS-based exfiltration.
- **Responder**: capture the SMB authentication the UNC path triggers.

## References

- MySQL Reference Manual: LOAD_FILE, `secure_file_priv`, FILE privilege
- OWASP Testing Guide: Testing for SQL Injection
