---
title: "Enumeration: mapping the vSphere inventory and services"
order: 1
description: "With access to vCenter, an attacker enumerates the full inventory through the vSphere API: every ESXi host, virtual machine, datastore, network, and user role. Unauthenticated, the exposed endpoints and version strings identify the build for matching known vulnerabilities. This reconnaissance scopes the estate and selects targets before any destructive action."
keywords:
  - vcenter enumeration
  - vsphere api
  - govc
  - inventory
  - version fingerprint
---

# Enumeration

vCenter's value to an attacker is its complete view of the environment, and the vSphere API exposes it. Authenticated, an attacker lists every host, VM, datastore, network, and permission in one place; unauthenticated, the exposed service endpoints and version strings fingerprint the build to match against known vulnerabilities. This reconnaissance maps the estate and picks targets, the high-value VMs, the datastores holding sensitive disks, the hosts to pivot through, before acting.

## Unauthenticated fingerprinting

```bash
# version and build, to match advisories
curl -sk https://<vcenter>/sdk/vimServiceVersions.xml
curl -sk https://<vcenter>/analytics/telemetry/ph/api/hyper/send 2>/dev/null
# the VAMI appliance management interface on 5480 and its version
curl -sk https://<vcenter>:5480/
```

## Authenticated inventory

```bash
# govc (vSphere CLI) against the API with credentials or a token
export GOVC_URL='https://<vcenter>' GOVC_USERNAME='administrator@vsphere.local' GOVC_PASSWORD='...' GOVC_INSECURE=1
govc about                                   # version/build
govc ls -l /                                 # datacenters, folders
govc find / -type h                          # every ESXi host
govc find / -type m                          # every VM
govc datastore.ls -l                         # datastores (disks to steal)
govc permissions.ls                          # roles and who holds them
```

## Exploitation notes

- The unauthenticated version string is the first thing to grab; vCenter bundles many services and its build maps directly to the applicable [known management exploits](known-management-exploits.md).
- Authenticated, the inventory selects targets: find domain controllers and sensitive servers among the VMs, and the datastores that hold their disks for offline theft.
- Permissions enumeration reveals which accounts hold administrative roles, guiding credential targeting and the SSO abuse routes.
- `govc` speaks the same API as the UI, so a stolen token or credential drives full enumeration non-interactively.

## Tools

- [govc (vSphere CLI)](https://github.com/vmware/govmomi/tree/main/govc)

## References

- [vSphere Web Services API](https://developer.vmware.com/apis/vsphere-automation/latest/)
- [VMware vCenter documentation](https://docs.vmware.com/en/VMware-vSphere/index.html)
