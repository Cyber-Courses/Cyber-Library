---
title: "Management plane: XCP-ng, XenServer, and xapi"
description: "Abusing the Xen management plane: the xapi management API and pool model used by XCP-ng and Citrix XenServer, and the XenCenter client, to control resource pools of hosts and their VMs with recovered pool credentials or a reachable management endpoint."
keywords:
  - XCP-ng
  - XenServer
  - xapi
  - XenCenter
  - resource pool
---

# Management plane

XCP-ng and Citrix XenServer manage hosts in resource pools through the `xapi` service and its API, driven by the `xe` CLI and the XenCenter client. A pool has a coordinator (master) that controls every member host. Reaching xapi with pool credentials, or compromising the coordinator, is control of the whole pool and all its VMs.

```bash
# xapi API via the xe CLI against a pool coordinator
xe -s <coordinator> -u root -pw <pass> vm-list
xe -s <coordinator> -u root -pw <pass> pool-list
xe host-list                               # every host in the pool
```

## Exploitation notes

- The pool coordinator is the high-value target: its `root` and the pool secret reach every member host as root.
- xapi is reachable over HTTPS on the hosts; recovered pool credentials or a management-endpoint flaw grant full control.
- Pool compromise cascades to dom0 on every host, enabling [Disk and snapshot theft](disk-and-snapshot-theft.md) across the fleet.

## References

- [XCP-ng: xe CLI](https://docs.xcp-ng.org/management/)
- [Xen API (xapi)](https://xapi-project.github.io/)
