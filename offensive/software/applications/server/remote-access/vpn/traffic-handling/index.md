---
title: "Traffic handling: VPN leakage and policy gaps"
description: "Even a cryptographically sound VPN leaks when its traffic handling is misconfigured: DNS queries escaping the tunnel disclose activity and enable redirection, routes that fail to cover intended destinations send traffic in the clear, and split tunneling bridges the client between the protected network and the open internet, a path an attacker on the client's local network exploits."
keywords:
  - vpn leak
  - dns leak
  - route leak
  - split tunneling
  - policy
---

# Traffic handling

A VPN can be perfectly encrypted and still fail through how it routes and resolves traffic. DNS queries that escape the tunnel disclose the user's activity to the local network and the configured resolver, and allow an on-path attacker to redirect names. Routes that do not cover all intended destinations (or that a local attacker can override) send supposedly-protected traffic in the clear. And split tunneling, where only some traffic goes through the VPN, deliberately bridges the client between the internal network and the open internet, which an attacker on the client's local segment leverages to reach the internal network through the client. These are configuration and policy weaknesses, not crypto breaks.

## Subtopics

- **[DNS leak](dns-leak.md)**: queries escaping the tunnel.
- **[Route leak](route-leak.md)**: traffic bypassing the tunnel through routing gaps.
- **[Split tunneling](split-tunneling.md)**: bridging the client between networks.

## References

- [NIST SP 800-77: VPN policy](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
- [HackTricks: VPN](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
