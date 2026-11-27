---
title: "Redfish API: abusing the modern BMC REST API"
order: 5
description: "Redfish is a standardised REST API used by modern BMCs. Its resources and actions are governed by implementation-specific roles and privileges; sufficiently privileged accounts may manage power, boot settings, accounts, and virtual media."
keywords:
  - redfish
  - rest api
  - bmc
  - virtual media
  - power control
---

# Redfish API

Redfish is the DMTF-standardised REST/JSON management API used by many modern BMCs, including iDRAC, iLO, Supermicro controllers, and OpenBMC. It can coexist with legacy IPMI rather than universally replacing it. Redfish provides a discoverable, scriptable control plane, but every operation is subject to the authenticated identity's role and privileges, with additional OEM variation. Depending on those permissions, an account may read inventory, manage power or boot settings, administer BMC accounts, or attach virtual media. Do not equate basic API access with all of those capabilities.

```bash
# discover the service root and authenticate with authorized test credentials
curl -sS --cacert <ca.pem> https://<bmc>/redfish/v1/
rf_base='https://<bmc>'; rf_creds='<user>:<password>'
# enumerate instance URIs instead of assuming vendor-specific identifiers
curl -sS --cacert <ca.pem> -u "$rf_creds" "$rf_base/redfish/v1/Systems/" | jq -r '.Members[]."@odata.id"'
# retrieve a chosen system and discover its advertised action target and allowed values
curl -sS --cacert <ca.pem> -u "$rf_creds" "$rf_base<system-uri>" | jq '.Actions, .Boot, .VirtualMedia'
# invoke the exact reset target advertised by that resource
curl -sS --cacert <ca.pem> -u "$rf_creds" -X POST "$rf_base<advertised-reset-target>" \
  -H 'Content-Type: application/json' -d '{"ResetType":"ForceRestart"}'
```

## Exploitation notes

- Redfish standardizes schemas and discovery, but resource instance URIs, supported actions, allowable values, and required privileges vary. Traverse `@odata.id` links and use each resource's advertised action target rather than hardcoding a vendor path.
- A high-impact combination is virtual-media, boot-configuration, and system-control access, which may permit starting attacker-controlled code. Required privileges, reboot access, firmware policy, Secure Boot, and disk encryption determine whether this becomes host or data compromise.
- Credentials come from [defaults](bmc-default-credentials.md), [cracked RAKP hashes](ipmi-rakp-hash-disclosure.md), or a controller [exploit](idrac-and-ilo.md); some Redfish implementations have also had their own authentication-bypass flaws, so test the service root and version.
- The service root may expose basic version and resource links without authentication; verify the actual response rather than assuming detailed firmware information is anonymous.

## References

- [DMTF Redfish specification](https://www.dmtf.org/standards/redfish)
- [DMTF Redfish protocol specification: privilege model](https://redfish.dmtf.org/schemas/DSP0266_1.20.1.html)
- [HackTricks: Redfish/BMC](https://book.hacktricks.xyz/network-services-pentesting/623-udp-ipmi)
