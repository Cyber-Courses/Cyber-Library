---
title: "Fortinet FortiOS: FortiGate SSL-VPN path traversal and heap RCE"
order: 1
description: "Fortinet FortiGate SSL-VPN (FortiOS) has had pre-authentication flaws reached on the web portal: a path-traversal that reads arbitrary files including the session and credential data, and heap buffer overflows giving remote code execution. Both are unauthenticated and have been exploited at scale for initial access into the internal network."
keywords:
  - fortinet
  - fortios
  - fortigate
  - ssl-vpn
  - path traversal
---

# Fortinet FortiOS

FortiGate's SSL-VPN, part of FortiOS, has been a repeated initial-access target through pre-authentication flaws on the SSL-VPN web portal. Two classes dominate. A path-traversal vulnerability let an unauthenticated attacker read arbitrary files from the appliance, including session files and credential material, directly from the portal. And heap buffer overflows in the SSL-VPN web daemon gave unauthenticated remote code execution on the device. Because FortiGate is internet-facing and bridges to the internal network, both have been exploited at scale; the approach is to fingerprint the FortiOS build and match it to the applicable flaw.

```bash
# fingerprint FortiGate/FortiOS and the SSL-VPN portal
curl -skI https://<gateway>/
curl -sk 'https://<gateway>/remote/login' | grep -i fortinet
# path-traversal class: unauthenticated arbitrary file read via a crafted portal request
#   target session/credential files on the appliance to recover access
# heap-overflow class: a crafted SSL-VPN request overflows the web daemon -> RCE
# match the exact FortiOS build to the advisory for the specific flaw + request.
```

## Exploitation notes

- The path-traversal yields unauthenticated arbitrary file read; the high-value targets are session files and stored credentials, which turn a read primitive into authenticated VPN access or admin.
- The heap-overflow class is direct unauthenticated RCE on the appliance, the strongest outcome, giving control of the device and its route inside.
- Both are version-specific; fingerprint the FortiOS build precisely (portal resources, headers) and match the advisory, since the vulnerable endpoint and payload differ per flaw.
- Post-exploitation on the appliance yields configuration, cached credentials, and the internal-network pivot; FortiGate compromise has been a staple ransomware entry point.

## References

- [Fortinet PSIRT advisories](https://www.fortiguard.com/psirt)
- [CISA: FortiOS exploitation](https://www.cisa.gov/news-events/cybersecurity-advisories)
