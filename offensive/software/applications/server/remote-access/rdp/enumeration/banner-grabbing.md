---
title: "Banner grabbing: RDP version and identity disclosure"
description: "RDP does not send a plaintext banner, but its connection handshake leaks the host identity: the NTLM negotiation during security setup discloses the NetBIOS and DNS hostname, the domain, and the OS build number. That build number dates the system and maps it to the pre-authentication RDP vulnerabilities it may be exposed to."
keywords:
  - rdp banner
  - rdp-ntlm-info
  - os build
  - hostname domain
  - version detection
---

# Banner grabbing

RDP has no cleartext banner like SSH, but its handshake is informative. During connection setup the server negotiates security, and when the NTLM provider is involved it returns an NTLM challenge whose target-info fields disclose the machine's NetBIOS and DNS names, the domain (or workgroup), the DNS domain and forest, and the OS version and build number. That build number is the key fact: it dates the Windows version and patch era, which determines whether the host falls in the range of the wormable pre-authentication RDP bugs and which authentication behaviour to expect.

```bash
# extract host identity and build from the NTLM handshake
nmap -p3389 --script rdp-ntlm-info <target>
#  => Target_Name, NetBIOS_Domain_Name, DNS_Computer_Name, Product_Version (build)
# confirm the port is RDP and get service/version
nmap -p3389 -sV <target>
```

## Exploitation notes

- `rdp-ntlm-info` is the RDP equivalent of a banner: the `Product_Version` build number maps directly to the OS release and patch window, telling you whether [BlueKeep](../pre-authentication-flaws/bluekeep.md) or [DejaBlue](../pre-authentication-flaws/dejablue.md) could apply.
- The disclosed hostname and domain feed Active Directory enumeration and targeted password spraying (domain\user formats), and confirm whether the host is domain-joined.
- This works even when NLA is required, because the NTLM exchange happens during security negotiation before a desktop session; it is unauthenticated reconnaissance.
- Pair the build with [security settings](security-settings.md) to complete the posture before choosing an attack.

## References

- [nmap rdp-ntlm-info](https://nmap.org/nsedoc/scripts/rdp-ntlm-info.html)
- [HackTricks: RDP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
