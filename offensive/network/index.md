---
title: "Network: offensive techniques by transport medium"
description: "The network category covers offensive work against network protocols and infrastructure, split by medium into wired and wireless because access and tooling differ fundamentally between them."
keywords:
  - network attacks
  - wired network
  - wireless attacks
  - protocol abuse
  - network pivoting
---

# Network

The network category covers offensive work against the links, protocols, and infrastructure that move data between systems. It spans discovery and mapping, interception and manipulation of traffic, abuse of protocol behavior, and using a foothold on one segment to reach another. The target here is the network itself, not the hosts on it, which are covered under [software](../software/index.md).

## Why it is split this way

The two subcategories divide by transport medium, because how an attacker gains access to the traffic, and the equipment required, differ fundamentally:

- **Wired**: attacks that need a physical or logical foothold on a wired segment, including layer-2 manipulation (ARP, VLAN, spanning tree), rogue devices, and interception on switched networks.
- **Wireless**: attacks that exploit radio as the shared medium, including Wi-Fi association and key attacks, rogue access points, and abuse of other RF protocols, where proximity replaces a cable.

The split matters because the first problem in a network attack is reaching the traffic at all. On a wired network that means a port, a tap, or a man-in-the-middle position; on wireless it means being in range and defeating the link-layer protections. Once traffic is reachable, the protocol-abuse techniques often converge, but the access phase that precedes them is medium-specific, which is why the category branches there first.

## References

- [OWASP: Testing Network Infrastructure](https://owasp.org/www-project-web-security-testing-guide/)
- [MITRE ATT&CK: Network-based techniques](https://attack.mitre.org/)
