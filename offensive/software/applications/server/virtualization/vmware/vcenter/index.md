---
title: "vCenter: attacking the vSphere management plane"
description: "vCenter Server manages a fleet of ESXi hosts and is the central control plane of a vSphere environment. Compromising it yields control of every managed host and VM. The surface is enumeration of the inventory and services, the Single Sign-On token and identity system, and a history of pre-authentication management-service vulnerabilities."
keywords:
  - vcenter
  - vsphere
  - sso
  - management plane
  - esxi fleet
---

# vCenter

vCenter Server is the management plane for a vSphere environment: it inventories and controls many ESXi hosts and all their virtual machines through a web UI and API. Compromising vCenter is compromising the whole estate, because it can run commands in guests, access every datastore, and administer every host. The offensive surface is the management services themselves: enumerating the inventory and exposed endpoints, abusing the Single Sign-On token and identity system, and exploiting the recurring pre-authentication vulnerabilities in vCenter's many bundled services.

```bash
# vCenter exposes a web UI/API and several supporting services
nmap -p 443,5480,902,2012,2014 <vcenter>        # UI/API, VAMI, authd, vpxd services
curl -sk https://<vcenter>/ui/                   # vSphere client
curl -sk https://<vcenter>/sdk/vimServiceVersions.xml   # API version/discovery
```

## Subtopics

- **[Enumeration](enumeration.md)**: mapping the inventory, hosts, and services.
- **[SSO and token abuse](sso-and-token-abuse.md)**: the Single Sign-On identity and token system.
- **[Known management exploits](known-management-exploits.md)**: the recurring pre-auth service vulnerabilities.

## References

- [VMware vCenter Server documentation](https://docs.vmware.com/en/VMware-vSphere/index.html)
- [VMware security advisories](https://www.vmware.com/security/advisories.html)
- [vSphere Web Services API reference](https://developer.vmware.com/apis/vsphere-automation/latest/)
