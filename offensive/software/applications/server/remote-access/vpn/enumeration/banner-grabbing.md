---
title: "Banner grabbing: VPN version and product disclosure"
description: "VPN gateways disclose identifying detail: IKE implementations leak a vendor fingerprint and transform sets, PPTP returns a vendor and firmware string, and SSL-VPN portals reveal the product and build through the login page, HTTP headers, and static resources. That version is the key to matching an appliance to its pre-authentication exploit."
keywords:
  - vpn banner
  - ike fingerprint
  - sslvpn portal
  - version
  - fingerprint
---

# Banner grabbing

Once the protocol is known, version and product detail come from the service. IKE implementations respond to `ike-scan` with a vendor ID and the offered transform sets, fingerprinting the device and its supported crypto. PPTP's control connection returns a vendor name and firmware version. SSL-VPN portals on 443 are the richest: the login page branding, HTTP headers, cookie names, and static resource paths (CSS/JS filenames and versions) identify the product (FortiGate, Ivanti/Pulse, GlobalProtect, Citrix/NetScaler, Cisco ASA) and often the exact build. That build is decisive, because the appliance exploits are version-specific.

```bash
ike-scan -M <target>                           # vendor ID + transforms (IKE fingerprint)
nmap -p1723 --script pptp-version <target>     # PPTP vendor/firmware
curl -skI https://<target>/                    # Server header, cookies
curl -sk https://<target>/ | grep -iE 'fortigate|pulse|ivanti|globalprotect|netscaler|citrix|anyconnect|cisco'
# product-specific paths/resources reveal the build (e.g. versioned JS/CSS, login paths)
```

## Exploitation notes

- The SSL-VPN product and build are the key facts: they map directly to the [appliance pre-auth exploits](../ssl-vpn-appliances/index.md), so fingerprint the login portal's branding, headers, cookies, and versioned resources carefully.
- IKE vendor IDs and transform sets fingerprint the IPsec device and reveal weak crypto support and aggressive-mode availability, feeding the [IKE attacks](../protocols/ipsec-ike.md).
- Portals frequently expose the build in a resource path or a version string even when the login page is generic; diff static resource hashes/names against known releases where needed.
- Combine with [protocol detection](protocol-detection.md) for a complete fingerprint before attacking.

## References

- [ike-scan vendor fingerprinting](https://github.com/royhills/ike-scan)
- [HackTricks: VPN enumeration](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
