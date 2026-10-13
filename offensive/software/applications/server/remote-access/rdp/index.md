---
title: "RDP: attacking the Remote Desktop Protocol"
order: 2
description: "RDP on TCP 3389 is Windows's graphical remote-administration protocol and one of the most attacked internet-exposed services. The surface is enumeration of version and security settings, authentication attacks and NLA considerations, internet exposure and weak TLS, pre-authentication protocol RCE, and abuse of sessions, hijacking, shadowing, and device redirection."
keywords:
  - rdp
  - remote desktop protocol
  - port 3389
  - nla
  - terminal services
---

# RDP

RDP (Remote Desktop Protocol) is Windows's graphical remote-administration service, listening on TCP 3389, and it is among the most heavily attacked services on the internet: exposed RDP is a primary initial-access vector for ransomware. Its surface spans enumeration (version, encryption and NLA settings, the TLS certificate), authentication (brute force, spraying, credential stuffing, and the role of Network Level Authentication), exposure (internet-facing endpoints and weak TLS), pre-authentication protocol vulnerabilities (the wormable RCE class), and session abuse, taking over, shadowing, or using the device-redirection channels of RDP sessions.

```bash
nmap -p3389 -sV --script rdp-ntlm-info,rdp-enum-encryption <target>
# NLA/security posture and NTLM-leaked host/domain info
```

## Subtopics

- **[Enumeration](enumeration/index.md)**: version, security settings, and certificate.
- **[Authentication](authentication/index.md)**: brute force, spraying, stuffing, and NLA.
- **[Exposure](exposure/index.md)**: internet-facing RDP and weak TLS.
- **[Pre-authentication flaws](pre-authentication-flaws/index.md)**: wormable protocol RCE.
- **[Session abuse](session-abuse/index.md)**: hijacking, shadowing, and device redirection.

## References

- [MS-RDPBCGR protocol specification](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rdpbcgr/)
- [HackTricks: RDP (3389)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
