---
title: "Split tunneling: bridging the client between networks"
description: "Split tunneling sends only some traffic through the VPN and the rest directly to the internet, so the client is simultaneously on the protected internal network and the open internet. An attacker who compromises or sits on the client's local network uses that bridge to reach the internal network through the client, defeating the perimeter the VPN represents."
keywords:
  - split tunneling
  - bridge
  - client compromise
  - pivot
  - perimeter bypass
---

# Split tunneling

Split tunneling is a policy where only traffic destined for the corporate network goes through the VPN while everything else goes directly to the internet. It is convenient and saves bandwidth, but it makes the client a bridge: at the same time, the client holds a route into the protected internal network (through the tunnel) and is exposed to its local network and the open internet (directly). An attacker who compromises the client, or sits on its untrusted local network (home, café, hotel), uses that dual presence to reach the internal network through the client, turning a single endpoint into a pivot across the VPN perimeter that the VPN was meant to enforce.

```bash
# confirm split tunneling on the client: only some routes go via the tunnel
ip route                                        # internal subnets via tun, default via local gw
# an attacker on the client's local network (or who compromised the client) then:
#  - pivots through the client into the internal subnets it routes to over the tunnel
#  - reaches the client's directly-exposed services from the untrusted local side
```

## Exploitation notes

- The risk is the simultaneous dual presence: the client routes to internal resources over the tunnel while remaining reachable from its untrusted local network, so compromising the client (from that local side or via the internet it directly touches) yields a pivot into the internal network.
- Full-tunnel mode (all traffic through the VPN) removes this bridge; split tunnel is the exploitable policy, so confirm the client's routing shows a local default route alongside internal tunnel routes.
- The attack is a client-side pivot rather than a gateway attack: compromise or man-in-the-middle the client on its local network, then use its tunnel routes to reach inside.
- Combine with [route](route-leak.md) and [DNS leaks](dns-leak.md); split tunneling is the policy that most directly turns an endpoint into a perimeter-crossing foothold.

## References

- [NIST SP 800-77: tunneling policy](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
- [HackTricks: VPN](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
