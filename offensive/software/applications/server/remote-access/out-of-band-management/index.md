---
title: "Out-of-band management: attacking server BMCs below the OS"
description: "Out-of-band management gives remote control of a server's hardware independent of its OS, through the baseboard management controller (BMC) speaking IPMI and vendor stacks (iDRAC, iLO) with the Redfish API. The surface is devastating: IPMI's cipher 0 and RAKP flaws, near-universal default credentials, and vendor exploits grant power, console, and virtual-media control that compromises the host OS."
keywords:
  - out-of-band management
  - bmc
  - ipmi
  - idrac
  - ilo
---

# Out-of-band management

Out-of-band management controls a server's hardware remotely and independently of its operating system, through a baseboard management controller (BMC): a small always-on computer on the motherboard with its own network interface, reachable whether or not the host OS is running. BMCs speak IPMI and the vendor stacks (Dell iDRAC, HPE iLO) and expose the Redfish REST API. The security is notoriously poor, and the impact is total: a BMC controls host power, the physical console (KVM), and virtual media (mounting an attacker ISO), so compromising it compromises the host itself, below and around any OS defense. The surface is IPMI's protocol flaws, pervasive default credentials, and vendor controller exploits.

```bash
nmap -sU -p623 --script ipmi-version,ipmi-cipher-zero <target>   # IPMI/BMC on UDP 623
curl -sk https://<bmc>/redfish/v1/                               # Redfish API
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
