---
title: "Remote Access: attacking remote-administration services"
order: 8
description: "Remote-access services let administrators and users control systems over the network, and they are a primary target because they bridge directly to a shell, desktop, or internal network. The area covers SSH, RDP, VNC, Telnet, the Berkeley r-commands, VPN gateways, WinRM, out-of-band BMC management, and third-party desktop software, keyed by service then by aspect."
keywords:
  - remote access
  - ssh
  - rdp
  - vpn
  - remote administration
---

# Remote Access

Remote-access services exist to control a system from elsewhere, so compromising one is, by definition, a direct route to a shell, a desktop, or the internal network behind a gateway. That makes them among the most valuable targets on a network and the leading initial-access vector in practice. The services differ in protocol and era but share a recurring set of aspects: enumeration and fingerprinting, authentication (credentials, keys, trust), transport cryptography, protocol or implementation flaws, and abuse of the access once gained. The area is organised by service, with those aspects under each.

## Finding remote-access services

```bash
# one sweep across the common remote-access ports
nmap -sV -p22,23,512,513,514,3389,5900-5902,5985,5986,443,500,1723,623 <target>
nmap -sU -p500,4500,623 <target>               # IKE/IPsec and IPMI (UDP)
#  22 SSH  23 Telnet  512-514 r-cmds  3389 RDP  5900+ VNC  5985/6 WinRM
#  443 SSL-VPN/portal  500/4500 IPsec  1723 PPTP  623 IPMI/BMC
```

## Subtopics

- **[SSH](ssh/index.md)**: the encrypted remote shell, keys, crypto, trust, and tunnelling.
- **[RDP](rdp/index.md)**: the Windows remote desktop, NLA, pre-auth RCE, and session abuse.
- **[VNC](vnc/index.md)**: the RFB desktop, weak auth and cleartext.
- **[Telnet](telnet/index.md)**: the cleartext terminal and telnetd flaws.
- **[Shell protocols](shell-protocols/index.md)**: the Berkeley r-commands and host trust.
- **[VPN](vpn/index.md)**: remote-access gateways, protocols, and SSL-VPN appliance exploits.
- **[WinRM](winrm/index.md)**: Windows Remote Management and PowerShell Remoting.
- **[Out-of-band management](out-of-band-management/index.md)**: BMCs, IPMI, iDRAC/iLO, and Redfish.
- **[Desktop software](desktop-software/index.md)**: third-party remote-desktop tools.

## References

- [CISA: remote-access exploitation advisories](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [MITRE ATT&CK: remote services](https://attack.mitre.org/techniques/T1021/)
- [HackTricks: network services pentesting](https://book.hacktricks.xyz/network-services-pentesting)
