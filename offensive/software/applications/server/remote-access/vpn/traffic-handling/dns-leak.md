---
title: "DNS leak: queries escaping the VPN tunnel"
description: "When a VPN client sends DNS queries outside the tunnel, to the local or ISP resolver rather than the VPN's, it discloses the names the user resolves and lets a local or on-path attacker see and redirect that activity. DNS leaks reveal internal hostnames and user behaviour and enable name-based redirection despite the tunnel."
keywords:
  - dns leak
  - resolver
  - tunnel
  - redirection
  - leak test
---

# DNS leak

A DNS leak is when the VPN client resolves names through a resolver outside the tunnel, the local network's or the ISP's, instead of the VPN-provided resolver, usually because the client fails to override the system resolver or the OS falls back. The consequence is disclosure: anyone who sees those queries (the local network, the leaked resolver) learns the names the user looks up, which reveals internal hostnames the user is reaching and their general activity, defeating part of the VPN's confidentiality. It also enables redirection: an on-path attacker answering the leaked queries steers the user to attacker-controlled addresses despite the VPN.

```bash
# from the client's perspective, check which resolver is actually used on the tunnel
resolvectl status                               # per-link DNS; is the VPN resolver authoritative?
# observe leaked queries from the local network (on-path)
tcpdump -i eth0 -n udp port 53                  # queries going out the local interface, not the tunnel
# public leak-test services confirm which resolver the client appears to use
```

## Exploitation notes

- For an attacker on the client's local network, leaked queries are both intelligence (the internal names and services the user reaches over the VPN) and an opportunity: answer the leaked DNS to redirect the user to attacker infrastructure.
- Internal hostnames in leaked queries map the protected network's naming and services, useful reconnaissance even without redirection.
- Leaks commonly arise from the client not setting itself as the authoritative resolver, IPv6 DNS escaping an IPv4-only tunnel, or OS resolver fallback; check per-link DNS on the client.
- This pairs with [route leak](route-leak.md) and [split tunneling](split-tunneling.md) as the traffic-handling gaps a local attacker exploits around an otherwise-sound tunnel.

## References

- [DNS leak background](https://www.dnsleaktest.com/what-is-a-dns-leak.html)
- [HackTricks: VPN leaks](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
