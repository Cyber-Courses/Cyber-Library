---
title: "Management plane: libvirt, oVirt, and RHV"
description: "Abusing the management plane over KVM hosts: remote libvirt connections, and the oVirt and Red Hat Virtualization engine that manages many KVM hosts through a web and REST interface, to control VMs across the estate with recovered or weak credentials."
keywords:
  - libvirt
  - oVirt
  - RHV
  - management engine
  - KVM management
---

# Management plane

Beyond a single host, KVM fleets are managed by remote libvirt and by oVirt or Red Hat Virtualization (RHV). The oVirt engine is a web and REST appliance that controls many hosts, holds their credentials, and runs VMs anywhere in the datacenter. Compromising it, or a remote libvirt endpoint, is control of the estate rather than one host.

```bash
# Remote libvirt control
virsh -c qemu+ssh://user@<host>/system list --all
# oVirt/RHV REST API with recovered credentials
curl -sk -u 'admin@internal:pass' https://<engine>/ovirt-engine/api/vms
```

## Exploitation notes

- The oVirt engine is the high-value target: its admin (`admin@internal`) and database reach every managed host and VM; newer versions issue an SSO bearer token from /ovirt-engine/sso/oauth/token rather than accepting basic auth.
- Remote libvirt over `qemu+tcp` with weak or no authentication is a direct network control path; `qemu+ssh` reuses SSH trust.
- Engine compromise yields host root through the management agent (VDSM) on each host.

## References

- [oVirt documentation](https://www.ovirt.org/documentation/)
- [libvirt: remote access](https://libvirt.org/remote.html)
