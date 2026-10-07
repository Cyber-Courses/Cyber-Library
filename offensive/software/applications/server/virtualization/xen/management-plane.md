---
title: "Management plane: the Xen toolstack and XAPI"
order: 3
description: "Xen is managed by a toolstack: xl/libxl on upstream Xen, and XAPI on XenServer and XCP-ng, which adds a pool-wide XML-RPC/JSON-RPC API on port 443. Access to XAPI controls every host and VM in a pool, VM lifecycle, console, disk operations, and host commands, so XAPI credentials, session tokens, or an API flaw give pool-wide control."
keywords:
  - xapi
  - xenserver
  - xcp-ng
  - toolstack
  - xe
---

# Management plane

Upstream Xen is driven locally from dom0 with the `xl` toolstack, but the managed distributions, XenServer and XCP-ng, add XAPI, a pool-wide management service exposing an XML-RPC/JSON-RPC API (on 443) and driven by the `xe` CLI. A pool is a set of hosts with a coordinator (master); XAPI on the coordinator controls every host and VM in the pool. Reaching XAPI with credentials, a stolen session token, or through an API flaw is pool-wide control: VM lifecycle, console access, disk and snapshot operations, and host-level commands, without needing a guest escape.

## Reach and drive XAPI

```bash
# the xe CLI against the pool coordinator
xe -s <host> -u root -pw <pw> vm-list
xe -s <host> -u root -pw <pw> host-list
# the API is XML-RPC/JSON-RPC over HTTPS; a session is opened then reused
curl -sk https://<host>/jsonrpc -d '{"method":"session.login_with_password","params":["root","<pw>"],"id":0,"jsonrpc":"2.0"}'
# act: run on a host, or operate on VM disks
xe -s <host> -u root -pw <pw> vm-import / vm-export ...   # export a VM (disk theft)
```

## Control actions

```text
With XAPI/pool access an attacker can:
- control every VM across the pool (start/stop/console/migrate)
- export or snapshot VM disks (VDIs) for offline data theft
- run host-level operations through host.* calls
- read pool configuration, users, and secrets
Compromising the pool coordinator extends control to all member hosts.
```

## Exploitation notes

- XAPI is pool-wide: compromising the coordinator's XAPI controls every host and VM in the pool, so it is the highest-value Xen management target.
- Session tokens are bearer credentials once `session.login_with_password` succeeds; a stolen token acts without re-authenticating, and `vm-export`/VDI operations pull guest disks without a guest escape.
- `xl`/libxl on upstream Xen is local to dom0 (no network API), so there the management route is dom0 access itself, see [Host access and shell](host-access-and-shell.md).
- Fingerprint XenServer/XCP-ng and XAPI versions for any management-plane advisory.

## References

- [XAPI and the xe CLI (XCP-ng/XenServer)](https://docs.xcp-ng.org/)
- [Xen Project toolstacks](https://wiki.xenproject.org/wiki/Choice_of_Toolstacks)
