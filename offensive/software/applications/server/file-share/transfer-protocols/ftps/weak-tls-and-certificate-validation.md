---
title: "Weak TLS and certificate validation: intercepting FTPS"
description: "Abusing weak FTPS transport security: clients that do not validate the server certificate, self-signed certificates accepted blindly, and deprecated ciphers or protocol versions, allowing an attacker in the path to intercept the session and recover the credentials and files it carries."
keywords:
  - FTPS TLS
  - certificate validation
  - self-signed
  - weak ciphers
  - man-in-the-middle
---

# Weak TLS and certificate validation

FTPS only protects the session if the client validates the server's certificate and negotiates strong TLS. Many FTPS clients and scripts skip certificate validation or accept self-signed certificates, so an attacker in the network path presents their own certificate and intercepts the session, recovering the credentials and files. Deprecated ciphers and protocol versions allow a downgrade to a breakable or strippable channel.

```bash
# Enumerate the FTPS TLS configuration (protocols, ciphers, cert)
nmap -p 21 --script ssl-enum-ciphers,ftp-syst <target>
sslscan --starttls=ftp <target>:21
```

## Exploitation notes

- Clients that ignore certificate errors (common in automation and legacy GUIs) are interceptable with a self-signed cert on a man-in-the-middle position.
- Support for SSLv3, TLS 1.0, RC4, or 3DES flags a downgradeable or weak channel.
- Explicit FTPS (AUTH TLS) can sometimes be stripped if the client does not require encryption, falling back to cleartext FTP.

## References

- [Nmap ssl-enum-ciphers](https://nmap.org/nsedoc/scripts/ssl-enum-ciphers.html)
- [RFC 4217: Securing FTP with TLS](https://www.rfc-editor.org/rfc/rfc4217)
