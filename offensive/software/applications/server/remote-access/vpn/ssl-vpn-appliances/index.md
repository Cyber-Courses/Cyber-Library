---
title: "SSL-VPN appliances: pre-authentication exploits on remote-access gateways"
order: 6
description: "SSL-VPN and remote-access gateway appliances are internet-facing, hold a route into the internal network, and run complex closed firmware, making their pre-authentication vulnerabilities the leading initial-access vector. The major products, Fortinet FortiOS, Ivanti Connect Secure, Palo Alto GlobalProtect, Citrix Gateway, and Cisco ASA, have each had unauthenticated exploit chains used at scale."
keywords:
  - ssl-vpn
  - appliance
  - pre-auth
  - initial access
  - gateway
---

# SSL-VPN appliances

SSL-VPN and remote-access gateway appliances concentrate risk: they are internet-facing by design, they hold a route into the internal network, and they run large, closed firmware stacks that have proven rich in vulnerabilities. The result is that their pre-authentication flaws, path traversal, authentication bypass, command injection, and memory corruption reachable on the web portal, are the single most consequential initial-access vector of recent years, exploited at scale by ransomware and state actors. Each major product has had unauthenticated chains: Fortinet FortiOS, Ivanti Connect Secure, Palo Alto GlobalProtect, Citrix Gateway, and Cisco ASA. The method is always to fingerprint the product and build, then match it to its known pre-auth exploit.

```bash
curl -skI https://<gateway>/                   # product/headers
curl -sk https://<gateway>/ | grep -iE 'fortigate|ivanti|pulse|globalprotect|netscaler|citrix|anyconnect'
```

## Subtopics

- **[Fortinet FortiOS](fortinet-fortios.md)**: FortiGate SSL-VPN path traversal and heap RCE.
- **[Ivanti Connect Secure](ivanti-connect-secure.md)**: auth-bypass plus command-injection chains.
- **[Palo Alto GlobalProtect](palo-alto-globalprotect.md)**: GlobalProtect/PAN-OS unauthenticated RCE.
- **[Citrix Gateway](citrix-gateway.md)**: traversal-to-RCE and the Citrix Bleed token disclosure.
- **[Cisco ASA and AnyConnect](cisco-asa-and-anyconnect.md)**: WebVPN traversal and portal credential attacks.

## References

- [CISA Known Exploited Vulnerabilities catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [CISA: edge-device exploitation advisories](https://www.cisa.gov/news-events/cybersecurity-advisories)
