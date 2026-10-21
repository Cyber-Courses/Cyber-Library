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

FTPS's security is only as good as its TLS and the policies enforced by both endpoints. Obsolete TLS versions and weak ciphers expand the attack surface, but server support alone does not mean an attacker can force or decrypt a session. A client that does not verify the server certificate can accept an attacker's certificate, enabling a man-in-the-middle from a suitable network position. Explicit FTPS begins in cleartext; suppressing or rejecting `AUTH TLS` causes exposure only when the client tolerates continuing without TLS. These are distinct findings with different prerequisites and impact.

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
- Weak-cipher or obsolete-protocol acceptance is a configuration finding; `ssl-enum-ciphers` grades what the server allows, but practical decryption depends on the negotiated suite and the requirements of its known attacks.
- The payoff is the same as plain FTP interception, captured credentials (often reusable system accounts) and file contents; the FTP-level issues (anonymous, brute force) remain regardless of TLS.

## References

- [RFC 4217: securing FTP with TLS](https://datatracker.ietf.org/doc/html/rfc4217)
- [RFC 8996: deprecating TLS 1.0 and TLS 1.1](https://www.rfc-editor.org/rfc/rfc8996.html)
- [nmap ssl-enum-ciphers](https://nmap.org/nsedoc/scripts/ssl-enum-ciphers.html)
