---
title: "Weak TLS and certificate validation: intercepting FTPS"
description: "FTPS protects FTP only when its TLS is strong and the client verifies the server certificate. Servers offering obsolete protocol versions or weak ciphers, allowing a plaintext fallback, or clients that skip certificate validation let an attacker in a network position downgrade or man-in-the-middle the connection, recovering the credentials and data FTPS was meant to protect."
keywords:
  - ftps tls
  - certificate validation
  - downgrade
  - man in the middle
  - starttls
---

# Weak TLS and certificate validation

FTPS's security is only as good as its TLS, and several common weaknesses undo it for an attacker with a network position. A server offering obsolete TLS versions or weak ciphers can be forced into a breakable session. A client that does not verify the server certificate accepts an attacker's certificate, enabling a straightforward man-in-the-middle. And explicit FTPS that permits a plaintext fallback (or whose `AUTH TLS` can be stripped) lets a downgrade attack return the session to cleartext. Any of these recovers the credentials and file data that FTPS is supposed to encrypt.

```bash
# assess the server's TLS posture
nmap -p990,21 --script ssl-enum-ciphers <target>     # weak protocols/ciphers offered?
openssl s_client -connect <target>:990 -tls1          # does it accept obsolete TLS?
# explicit FTPS downgrade: strip AUTH TLS so the client stays plaintext (MITM position)
#   intercept the control channel and remove/deny the AUTH TLS capability/command
# certificate check: a self-signed or unverified cert accepted by the client => MITM
openssl s_client -connect <target>:990 | openssl x509 -noout -issuer -subject
```

## Exploitation notes

- The attack needs a network position (on-path or via name-resolution poisoning); from there, a client that skips certificate validation is trivially intercepted with an attacker-presented certificate.
- Explicit FTPS (`AUTH TLS` on 21) is the more interceptable variant because the session begins in cleartext; stripping or denying the TLS upgrade keeps it plaintext if the client tolerates it.
- Weak-cipher or obsolete-protocol acceptance enables decrypting an otherwise-TLS session; `ssl-enum-ciphers` grades what the server allows.
- The payoff is the same as plain FTP interception, captured credentials (often reusable system accounts) and file contents; the FTP-level issues (anonymous, brute force) remain regardless of TLS.

## References

- [RFC 4217: securing FTP with TLS](https://datatracker.ietf.org/doc/html/rfc4217)
- [nmap ssl-enum-ciphers](https://nmap.org/nsedoc/scripts/ssl-enum-ciphers.html)
