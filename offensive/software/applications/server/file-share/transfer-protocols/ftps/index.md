---
title: "FTPS: attacking FTP over TLS"
description: "FTPS adds TLS to FTP, either implicitly on port 990 or explicitly via AUTH TLS on port 21. It protects the credentials and data that plain FTP exposes, but only when the TLS is configured and verified correctly. The offensive surface is weak or unverified TLS allowing interception, and the underlying FTP weaknesses (anonymous access, brute force) that TLS does not remove."
keywords:
  - ftps
  - auth tls
  - port 990
  - tls
  - ftp over tls
---

# FTPS

FTPS is FTP wrapped in TLS. It comes in two forms: implicit FTPS, which negotiates TLS immediately on connection to port 990, and explicit FTPS, which starts plaintext on port 21 and upgrades with the `AUTH TLS` command. Done correctly it fixes FTP's biggest flaw, the cleartext credentials and data, but the protection depends entirely on the TLS being configured and the client verifying it. The offensive surface is therefore twofold: weak or unverified TLS that still permits interception, and the FTP-level weaknesses (anonymous access, brute force, bounce) that encryption does not address.

```bash
# detect FTPS and inspect its TLS
nmap -p21,990 --script ftp-syst,ssl-enum-ciphers <target>
openssl s_client -connect <target>:990           # implicit FTPS
# explicit: connect plain to 21, then AUTH TLS, then STARTTLS handshake
curl -v --ssl-reqd ftp://<target>/ --user user:pass
```

## Subtopics

- **[Weak TLS and certificate validation](weak-tls-and-certificate-validation.md)**: interception through poor TLS configuration.

## References

- [RFC 4217 (Securing FTP with TLS)](https://datatracker.ietf.org/doc/html/rfc4217)
- [HackTricks: FTP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp)
