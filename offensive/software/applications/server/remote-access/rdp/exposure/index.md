---
title: "Exposure: internet-facing RDP and weak transport"
description: "RDP is routinely exposed directly to the internet on 3389, where it is continuously scanned, brute-forced, and stuffed, and it is a leading ransomware entry point. Exposure analysis finds these endpoints and assesses the transport: native RDP security and weak TLS allow machine-in-the-middle and decryption, compounding the credential and pre-auth risks."
keywords:
  - rdp exposure
  - internet-facing
  - port 3389
  - weak tls
  - attack surface
---

# Exposure

RDP's biggest real-world risk is simple exposure: organizations publish 3389 to the internet for convenience, and those endpoints are perpetually scanned, brute-forced, and credential-stuffed, making exposed RDP a top ransomware initial-access vector. Exposure analysis has two parts: finding the internet-facing endpoints (directly or via search engines like Shodan), and assessing the transport security, because a host using legacy native RDP security or weak TLS is additionally open to machine-in-the-middle and session decryption, which compounds the already-high credential and pre-auth exposure.

```bash
# find exposed RDP
nmap -p3389 --open <range>
# search-engine discovery: shodan "port:3389", masscan for breadth
# assess transport
nmap -p3389 --script rdp-enum-encryption,ssl-enum-ciphers <target>
```

## Subtopics

- **[Internet exposure](internet-exposure.md)**: finding and assessing internet-facing RDP.
- **[Weak TLS](weak-tls.md)**: native RDP security and TLS weaknesses enabling MITM.

## References

- [CISA: RDP exposure and ransomware](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [HackTricks: RDP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
