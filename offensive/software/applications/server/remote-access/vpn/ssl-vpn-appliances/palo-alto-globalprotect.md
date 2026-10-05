---
title: "Palo Alto GlobalProtect: unauthenticated RCE on the gateway"
description: "Palo Alto GlobalProtect and the PAN-OS management interface have had unauthenticated remote code execution flaws, including command injection reachable on the GlobalProtect portal and gateway. Exploited for initial access, they give code execution as a privileged account on the firewall, which fronts and routes into the internal network."
keywords:
  - palo alto
  - globalprotect
  - pan-os
  - command injection
  - unauthenticated rce
---

# Palo Alto GlobalProtect

GlobalProtect is Palo Alto's SSL-VPN, served by the PAN-OS firewall, and both the GlobalProtect portal/gateway and the PAN-OS management interface have carried unauthenticated remote code execution flaws. The recurring class is command injection, where attacker-controlled input reaches an OS command on the device without authentication, giving code execution as a privileged account. Because the GlobalProtect portal is internet-facing and the firewall routes into the internal network, these have been used for initial access at scale; some chains combine an injection with a path or validation flaw to reach the vulnerable sink unauthenticated.

```bash
# fingerprint GlobalProtect/PAN-OS
curl -skI https://<gateway>/
curl -sk 'https://<gateway>/global-protect/login.esp' | grep -i 'globalprotect\|pan-os'
# command-injection class: a crafted value in a GlobalProtect/management request reaches
# an OS command unauthenticated -> code execution as a privileged device account.
# match the PAN-OS/GlobalProtect version to the advisory for the specific injection + path.
```

## Exploitation notes

- Command injection on the GlobalProtect portal or PAN-OS management is the dominant class and gives direct unauthenticated code execution on the firewall as a privileged account.
- Some chains pair the injection with a path-traversal or validation flaw to reach the sink without authentication; identify both halves for the target build.
- The firewall is the perimeter: appliance code execution yields configuration, credentials, and the internal pivot, and persistence on the device survives beyond a session.
- Version-specific; fingerprint PAN-OS/GlobalProtect (the `global-protect` portal resources) and match the advisory for the exact flaw.

## References

- [Palo Alto Networks Security Advisories](https://security.paloaltonetworks.com/)
- [CISA: PAN-OS/GlobalProtect exploitation](https://www.cisa.gov/news-events/cybersecurity-advisories)
