---
title: "Route leak: traffic bypassing the VPN through routing gaps"
description: "A VPN protects only the traffic its routes send through the tunnel. When the pushed routes do not cover all intended destinations, when IPv6 is unrouted on an IPv4 tunnel, or when a local attacker injects a more specific route, traffic meant for the tunnel goes out in the clear, exposing it to the local network despite the VPN being up."
keywords:
  - route leak
  - routing table
  - ipv6 leak
  - more specific route
  - tunnel bypass
---

# Route leak

A VPN only protects traffic that its routing sends into the tunnel; anything the routing table directs elsewhere leaves in the clear. Route leaks happen several ways: the server pushes routes that do not actually cover all the destinations the user believes are protected; IPv6 traffic is left on the native interface when the tunnel only captures IPv4 (so dual-stack destinations leak over IPv6); or a local attacker injects a more specific route (via rogue router advertisements or DHCP) that wins over the VPN's route, pulling targeted traffic out of the tunnel. In each case, supposedly-protected traffic is exposed to the local network while the VPN appears up.

```bash
# inspect the client routing table to see what is (not) tunnelled
ip route; ip -6 route                           # does the default/intended route go via the tun?
#   a native-interface route for a destination means it bypasses the tunnel
# local attacker: inject a more-specific route to pull traffic out of the tunnel
#   rogue RA (IPv6) or DHCP option to add a competing, more-specific route on the client
```

## Exploitation notes

- IPv6 is the classic leak: an IPv4-only tunnel on a dual-stack client sends IPv6 traffic natively, so an attacker offering IPv6 connectivity (rogue router advertisements) captures that traffic outside the VPN.
- Longest-prefix-match routing means a more-specific route injected by a local attacker (rogue RA/DHCP) beats the VPN's broader route for the targeted destination, pulling it into the clear, a selective bypass.
- Incomplete pushed routes expose whatever they omit; compare the client's routing table against what the user assumes is protected.
- Route leaks expose the traffic content (not just DNS); combine with a capture position and with [DNS leak](dns-leak.md) and [split tunneling](split-tunneling.md) for the full traffic-handling picture.

## References

- [IPv6 VPN leakage](https://www.rfc-editor.org/rfc/rfc7359)
- [HackTricks: VPN leaks](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
