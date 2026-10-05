---
title: "Security settings: RDP NLA, encryption, and certificate posture"
description: "RDP's security settings decide how it can be attacked: whether Network Level Authentication is required (forcing credentials before a session), which security layer and encryption level are negotiated (native RDP security versus TLS), and the server certificate. Enumerating these separates hardened hosts from those open to unauthenticated connection and pre-auth exploitation."
keywords:
  - nla
  - encryption level
  - security layer
  - tls certificate
  - rdp configuration
---

# Security settings

How an RDP host is configured determines which attacks are even possible, so enumerating its security settings is decisive. Three things matter. Network Level Authentication (NLA, using CredSSP) requires the client to authenticate before any session is created; with NLA off, a client reaches the logon surface (and historically the vulnerable pre-auth code) without credentials. The security layer and encryption level reveal whether the host uses legacy native RDP security (weak) or TLS, and how strong. And the TLS certificate names the host and dates it. Together these separate a hardened, NLA-enforcing, TLS host from a legacy, NLA-off host open to unauthenticated connection and pre-auth bugs.

```bash
# security layer, encryption level, and NLA requirement
nmap -p3389 --script rdp-enum-encryption <target>
#   reports: Security layer (RDP/SSL/CredSSP/Hybrid), Encryption level, NLA supported/required
# inspect the TLS certificate (host identity, validity, key strength)
openssl s_client -connect <target>:3389 -starttls rdp 2>/dev/null | openssl x509 -noout -subject -dates
```

## Exploitation notes

- NLA status is the pivotal finding: NLA required means credentials are needed before a session, which blunts pre-auth exploitation and makes brute force the path; NLA not required exposes the pre-auth surface and lets a client reach the logon screen unauthenticated.
- The security layer tells the crypto story: native "RDP Security" (RC4-based) is weak and MITM-prone, while CredSSP/TLS is stronger; a legacy layer flags an old, likely-unpatched host.
- The certificate's self-signed default, subject (hostname), and validity dates corroborate the OS age and host identity; a long-expired or default cert signals neglect.
- Combine with the [build number](banner-grabbing.md): an old build plus NLA-off is a prime [pre-auth RCE](../pre-authentication-flaws/index.md) target; a modern build with NLA required routes you to [authentication](../authentication/index.md) and [exposure](../exposure/index.md).

## References

- [nmap rdp-enum-encryption](https://nmap.org/nsedoc/scripts/rdp-enum-encryption.html)
- [Microsoft: Network Level Authentication](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/)
