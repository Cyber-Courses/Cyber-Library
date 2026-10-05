---
title: "FTPS: attacking FTP over TLS"
description: "Attacking FTPS, FTP wrapped in TLS: weak or unvalidated certificates that permit interception, deprecated ciphers and protocol versions that allow downgrade, and the same anonymous and brute-force weaknesses as plain FTP underneath the transport layer."
keywords:
  - FTPS
  - FTP over TLS
  - certificate validation
  - weak ciphers
  - downgrade
---

# FTPS

FTPS adds TLS to FTP's control and data channels. Its weaknesses are the TLS configuration (unvalidated or self-signed certificates that enable man-in-the-middle, and deprecated ciphers and protocol versions that permit downgrade) layered over the same anonymous-access and brute-force issues as plain FTP.

## Subtopics

- **[Weak TLS and certificate validation](weak-tls-and-certificate-validation.md)**: interception and downgrade.

## References

- [HackTricks: pentesting FTP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ftp/index.html)
- [RFC 4217: Securing FTP with TLS](https://www.rfc-editor.org/rfc/rfc4217)
