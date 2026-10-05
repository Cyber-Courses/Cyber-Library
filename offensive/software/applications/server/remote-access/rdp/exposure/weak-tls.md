---
title: "Weak TLS: RDP transport weaknesses enabling MITM"
description: "RDP can use legacy native RDP security (RC4-based, with a known MITM weakness) or TLS that may be misconfigured with weak ciphers and an untrusted self-signed certificate. A positioned attacker exploits the native-security weakness or weak/unverified TLS to machine-in-the-middle the connection, capturing credentials and session content."
keywords:
  - rdp tls
  - native rdp security
  - rc4
  - mitm
  - self-signed certificate
---

# Weak TLS

RDP's transport security varies, and the weak configurations enable machine-in-the-middle. Legacy "native RDP security" uses RC4 with a design weakness that allowed a classic RDP MITM (the server's public key is not authenticated, so an attacker interposes). The modern option is TLS, but it is frequently weakened: default self-signed certificates that clients are trained to accept, obsolete protocol versions, and weak cipher suites. Either way, a positioned attacker who exploits native-security's lack of server authentication, or a client that does not verify the TLS certificate, interposes to capture credentials and the session.

```bash
# determine the security layer and TLS posture
nmap -p3389 --script rdp-enum-encryption <target>      # native RDP security vs TLS/CredSSP
nmap -p3389 --script ssl-enum-ciphers <target>         # weak ciphers/protocols if TLS
openssl s_client -connect <target>:3389 -starttls rdp 2>/dev/null | openssl x509 -noout -issuer -subject
# with an on-path position, run an RDP MITM that presents a substitute cert/key
#   (tools like Seth automate RDP downgrade + MITM against native security / unverified TLS)
```

## Exploitation notes

- Native "RDP Security" is the strongest finding here: its RC4-based layer does not authenticate the server, so an on-path attacker performs a full MITM, downgrade-and-intercept, capturing the cleartext credentials entered at logon.
- For TLS-protected RDP, the weakness is usually trust: default self-signed certs that users click through, enabling an attacker-presented certificate; weak cipher/protocol acceptance is a secondary concern.
- The practical payoff of an RDP MITM is credential capture (the user authenticates to the attacker) plus session observation; it needs a network position (ARP/DNS/route).
- Enumerate the security layer first ([security settings](../enumeration/security-settings.md)); native security or unverified TLS is what makes the MITM viable.

## Tools

- [Seth (RDP MITM)](https://github.com/SySS-Research/Seth)

## References

- [SySS: RDP MITM research](https://www.syss.de/en/)
- [Microsoft: RDP security layers](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/)
