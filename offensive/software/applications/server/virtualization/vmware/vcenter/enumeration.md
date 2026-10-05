---
title: "Enumeration: mapping a vCenter inventory and identities"
description: "Enumerating a VMware vCenter after reaching it: inventorying the managed ESXi hosts, VMs, datastores, and permissions through the vSphere API, and reading the SSO identity sources and administrators to plan escalation to full control."
keywords:
  - vCenter enumeration
  - vSphere API
  - govc
  - SSO identity
  - inventory
---

# Enumeration

With any vCenter credential, the API maps the estate: every managed host, VM, datastore, resource pool, and permission. Even a low-privileged account reveals the attack surface and often datastore or guest access, and the SSO configuration shows which identities hold administrator.

```bash
export GOVC_URL='https://user:pass@vcenter' GOVC_INSECURE=1
govc ls -l /                                  # datacenters, hosts, VMs
govc host.info; govc vm.info -all '*'
govc permissions.ls /                         # who can do what
govc datastore.ls -l                          # datastores (VMDK access)
```

## Exploitation notes

- Inventory reveals domain controller VMs and other high-value guests to target for [Datastore and VMDK theft](../esxi/datastore-and-vmdk-theft.md).
- Permissions and SSO groups show the path to administrator; the `vsphere.local` SSO domain and `Administrators` group are the goal.
- Low-privilege API access frequently still allows console or datastore browsing, enough to pivot without full admin.

## References

- [govc CLI](https://github.com/vmware/govmomi/tree/main/govc)
- [pyVmomi SDK](https://github.com/vmware/pyvmomi)
