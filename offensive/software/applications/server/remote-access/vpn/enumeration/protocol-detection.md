---
title: "Protocol detection: identifying the VPN protocol and ports"
description: "Different VPN technologies listen on characteristic ports and respond to distinct probes: IPsec/IKE on UDP 500 and 4500, PPTP on TCP 1723, OpenVPN typically on 1194, SSTP and SSL-VPN portals on TCP 443, and WireGuard on a UDP port. Identifying the protocol selects the applicable attack and tooling."
keywords:
  - protocol detection
  - port 500
  - port 1723
  - openvpn
  - sslvpn
---

# Protocol detection

Each VPN technology has a recognisable footprint, and identifying it is the first enumeration step because the protocol dictates the attack. IPsec uses UDP 500 for IKE and 4500 for NAT-traversal; PPTP uses TCP 1723 (with GRE); OpenVPN defaults to UDP or TCP 1194; SSTP and SSL-VPN portals ride TCP 443 (distinguished by the HTTP response and product); and WireGuard listens on a configurable UDP port, responding only to a valid handshake (silent otherwise). A UDP scan plus targeted probes maps the gateway to its protocol.

```bash
# scan the characteristic ports
nmap -sU -p500,4500,1194,51820 <target>        # IKE, NAT-T, OpenVPN(UDP), WireGuard
nmap -sT -p1723,443,1194 -sV <target>          # PPTP, SSL-VPN/SSTP, OpenVPN(TCP)
ike-scan <target>                               # confirms IPsec/IKE by handshake
# OpenVPN and WireGuard are often silent to non-protocol probes; a valid handshake confirms
```

## Exploitation notes

- Port 500/4500 UDP responding to `ike-scan` confirms IPsec and opens the [IKE](../protocols/ipsec-ike.md) attacks (aggressive mode, PSK); 1723 confirms [PPTP](../protocols/pptp.md); 443 with a VPN portal points at an [SSL-VPN appliance](../ssl-vpn-appliances/index.md).
- WireGuard and OpenVPN are often silent to generic scans (WireGuard replies only to a cryptographically valid handshake), so absence of a banner does not mean absence of the service; infer from the port and any client config.
- On 443, distinguish a plain SSL-VPN portal from SSTP and from a general web app by the HTTP response and product fingerprints, which [banner grabbing](banner-grabbing.md) refines.
- The identified protocol selects both the attack and the tooling (`ike-scan` for IPsec, product-specific exploits for appliances).

## References

- [ike-scan](https://github.com/royhills/ike-scan)
- [HackTricks: VPN protocols](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
