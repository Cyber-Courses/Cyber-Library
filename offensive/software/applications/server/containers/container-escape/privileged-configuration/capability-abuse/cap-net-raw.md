---
title: "CAP_NET_RAW: raw packet crafting and sniffing from a container"
description: "Abusing a container that holds CAP_NET_RAW to open raw and packet sockets, sniffing traffic reachable on its networks and spoofing or injecting packets to poison name resolution, impersonate services, or redirect traffic for a man-in-the-middle."
keywords:
  - CAP_NET_RAW
  - raw socket
  - packet sniffing
  - ARP spoofing
  - container network attack
---

# CAP_NET_RAW

`CAP_NET_RAW` allows raw and packet sockets. It is not a host escape but a network attack primitive: the container can sniff traffic on networks it shares and craft arbitrary packets to spoof, poison, or redirect. On a shared or host network it is especially dangerous.

```bash
capsh --print | grep -q cap_net_raw && echo have
tcpdump -i any -c 20 2>/dev/null                 # sniff reachable traffic
# Spoof responses: ARP poisoning, DHCP, or DNS depending on the segment
```

## Exploitation notes

- The impact scales with the network the container sits on; combine with a [Host network namespace](../../shared-host-namespaces/host-network-namespace.md) to reach the host's own interfaces and services.
- Classic uses are ARP and DNS spoofing to man-in-the-middle other pods or the node, and capturing credentials in cleartext protocols.
- In Kubernetes, raw sockets plus a flat pod network often reach services a NetworkPolicy was assumed to protect.

## References

- [man 7 raw](https://man7.org/linux/man-pages/man7/raw.7.html)
- [man 7 capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html)
