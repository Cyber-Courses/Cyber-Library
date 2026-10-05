---
title: "Internet exposure: finding and assessing internet-facing RDP"
description: "Directly exposing RDP on 3389 to the internet is a leading ransomware entry point. Attackers find these endpoints by mass scanning and search engines, fingerprint the host and its NLA/version posture, and then apply credential and pre-auth attacks. Non-standard ports and RD Gateway front-ends shift but do not remove the exposure."
keywords:
  - internet exposure
  - port 3389
  - shodan
  - rd gateway
  - ransomware
---

# Internet exposure

Publishing RDP straight to the internet is one of the most consequential misconfigurations in practice: exposed 3389 endpoints are continuously discovered and attacked, and they are a dominant initial-access route for ransomware. Finding them is trivial, mass port scanning and search engines index them, and once found the host is fingerprinted and then attacked through credentials or pre-auth bugs. Moving RDP to a non-standard port or fronting it with RD Gateway (RDP over HTTPS/443) changes the discovery method but does not remove the underlying exposure.

```bash
# direct discovery
nmap -p3389 --open <range>
masscan -p3389 <range> --rate 1000
# search-engine discovery (external): Shodan/Censys "port:3389" or RDP product tags
# RD Gateway front-end (RDP tunnelled over 443)
nmap -p443 --script http-title <target> | grep -i 'RD Web\|Remote Desktop'
# fingerprint discovered hosts for posture
nmap -p3389 --script rdp-ntlm-info,rdp-enum-encryption <discovered>
```

## Exploitation notes

- Discovery is easy and continuous; the point of this step is to inventory exposed hosts and immediately fingerprint their NLA status, version, and domain so the follow-on (credentials vs pre-auth) is chosen per host.
- Non-standard ports only obscure: a full-range `-sV` scan still identifies RDP, and search engines index it regardless of port.
- RD Gateway and RD Web Access move RDP onto 443, which is a different surface (HTTPS, web auth, sometimes MFA) but still ultimately brokers RDP; enumerate those web front-ends separately.
- Internet-facing RDP combines every other RDP weakness: it is where [credential stuffing](../authentication/credential-stuffing.md), spraying, and [pre-auth RCE](../pre-authentication-flaws/index.md) are applied at scale.

## References

- [CISA: RDP exposure guidance](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [Microsoft: RD Gateway](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/)
