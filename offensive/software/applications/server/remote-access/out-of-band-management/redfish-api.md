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
# discover the service root and authenticate with authorized test credentials
curl -sk https://<bmc>/redfish/v1/
rf_base='https://<bmc>'; rf_creds='<user>:<password>'
# enumerate instance URIs instead of assuming vendor-specific identifiers
curl -sk -u "$rf_creds" "$rf_base/redfish/v1/Systems/" | jq -r '.Members[]."@odata.id"'
# retrieve a chosen system and discover its advertised action target and allowed values
curl -sk -u "$rf_creds" "$rf_base<system-uri>" | jq '.Actions, .Boot, .VirtualMedia'
# invoke the exact reset target advertised by that resource
curl -sk -u "$rf_creds" -X POST "$rf_base<advertised-reset-target>" \
  -H 'Content-Type: application/json' -d '{"ResetType":"ForceRestart"}'
```

## Exploitation notes

- Redfish standardizes schemas and discovery, but resource instance URIs, supported actions, allowable values, and required privileges vary. Traverse `@odata.id` links and use each resource's advertised action target rather than hardcoding a vendor path.
- The highest-impact action is the same as elsewhere, attach attacker virtual media and set a one-time boot to compromise the host OS; power/boot control and account creation support that and persistence.
- Credentials come from [defaults](bmc-default-credentials.md), [cracked RAKP hashes](ipmi-rakp-hash-disclosure.md), or a controller [exploit](idrac-and-ilo.md); some Redfish implementations have also had their own authentication-bypass flaws, so test the service root and version.
- The service root may expose basic version and resource links without authentication; verify the actual response rather than assuming detailed firmware information is anonymous.

## References

- [DMTF Redfish specification](https://www.dmtf.org/standards/redfish)
- [HackTricks: Redfish/BMC](https://book.hacktricks.xyz/network-services-pentesting/623-udp-ipmi)
