---
title: "Out-of-band management: attacking server BMCs below the OS"
order: 9
description: "Out-of-band management gives remote control of a server's hardware independent of its OS through a baseboard management controller (BMC). Exposed legacy protocols, unchanged credentials, vulnerable firmware, and overly privileged accounts can grant power, console, boot, or virtual-media control and create a path to host compromise."
keywords:
  - out-of-band management
  - bmc
  - ipmi
  - idrac
  - ilo
---

# Out-of-band management

Out-of-band management controls a server's hardware remotely and independently of its operating system through a baseboard management controller (BMC): a small, always-on computer on the motherboard with its own network interface, reachable whether or not the host OS is running. Depending on vendor, generation, and configuration, a BMC may expose IPMI, a vendor interface such as Dell iDRAC or HPE iLO, and Redfish. Compromise is high impact because sufficiently privileged accounts can control host power, the console (KVM), boot settings, and virtual media. Turning that access into host compromise still depends on the granted privileges and platform controls such as boot policy, Secure Boot, and disk encryption. The attack surface includes the RAKP protocol-design weakness, deployments that permit cipher suite zero, unchanged credentials, vulnerable firmware, and overly broad network exposure.

```bash
nmap -sU -p623 --script ipmi-version,ipmi-cipher-zero <target>   # IPMI/BMC on UDP 623
curl -sS --cacert <ca.pem> https://<bmc>/redfish/v1/             # Redfish API
```

## Subtopics

- **[IPMI cipher 0 bypass](ipmi-cipher-0-bypass.md)**: authentication-disabling cipher suite zero.
- **[IPMI RAKP hash disclosure](ipmi-rakp-hash-disclosure.md)**: pre-auth password-hash retrieval.
- **[BMC default credentials](bmc-default-credentials.md)**: shipped admin credentials.
- **[iDRAC and iLO](idrac-and-ilo.md)**: Dell and HPE controller exploits and virtual media.
- **[Redfish API](redfish-api.md)**: abusing the modern BMC REST API.

## References

- [IPMI 2.0 specification](https://www.intel.com/content/www/us/en/products/docs/servers/ipmi/ipmi-second-gen-interface-spec-v2-rev1-1.html)
- [DMTF Redfish](https://www.dmtf.org/standards/redfish)
- [HackTricks: IPMI (623)](https://book.hacktricks.xyz/network-services-pentesting/623-udp-ipmi)
