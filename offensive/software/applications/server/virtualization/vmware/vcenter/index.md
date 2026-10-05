---
title: "vCenter: attacking the VMware management plane"
description: "Attacking VMware vCenter, the appliance that manages fleets of ESXi hosts: enumerating the inventory, abusing the SSO and SAML token system to forge administrator access, and exploiting the known management-service vulnerabilities, any of which grants control of every managed host and VM."
keywords:
  - vCenter
  - vSphere SSO
  - SAML
  - vmdir
  - management plane
---

# vCenter

vCenter Server manages many ESXi hosts, so compromising it is compromising the whole virtual estate. It is a web and API appliance built on a photon-OS base with a single sign-on (SSO) system, an identity store (vmdir), and a history of critical management-service flaws. Control of vCenter yields the `vpxuser` credential for every host and the ability to run on any VM.

## Subtopics

- **[Enumeration](enumeration.md)**: mapping the inventory and identities.
- **[SSO and token abuse](sso-and-token-abuse.md)**: forging administrator access.
- **[Known management exploits](known-management-exploits.md)**: critical service vulnerabilities.

## References

- [VMware vCenter Server security](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
