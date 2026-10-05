---
title: "Management plane: the Proxmox API, web UI, and cluster"
description: "Proxmox is managed through a REST API and web UI on port 8006, authenticated with tickets or API tokens and authorized by a role system. Access controls every VM and container: console and guest-agent command execution, disk and backup operations, and node shell. Ticket, token, or privilege-escalation flaws in this surface give estate control without a guest escape."
keywords:
  - proxmox api
  - pveproxy
  - api token
  - ticket
  - pvesh
---

# Management plane

Proxmox's control surface is the REST API and web UI served by `pveproxy` on port 8006, with the `pvesh` CLI as a local client. Authentication uses a login ticket (a signed token in a cookie) or an API token, and authorization is a role/permission system over a path hierarchy. Reaching this surface with sufficient rights is estate control: open a VM console, run commands via the guest agent, enter containers, create and restore backups, attach disks, and open a node shell. Flaws in ticket or token handling, or privilege escalation within the role system, give that control without any guest escape.

## Drive the API

```bash
# authenticate for a ticket, or use an API token directly
curl -sk -d 'username=root@pam&password=<pw>' https://<node>:8006/api2/json/access/ticket
# API token form (no login needed)
H='Authorization: PVEAPIToken=user@realm!tokenid=<secret>'
curl -sk -H "$H" https://<node>:8006/api2/json/cluster/resources   # all VMs/cts/nodes
# act: run a command inside a VM via the guest agent
curl -sk -H "$H" -XPOST https://<node>:8006/api2/json/nodes/<node>/qemu/<vmid>/agent/exec \
  -d 'command=id'
# or enter a container / open a node shell through the API
```

## Control actions

```text
With management access an attacker can:
- execute in VMs via the guest agent, and in containers via pct/exec endpoints
- create/restore vzdump backups and attach/clone disks (data access, see disk theft)
- open the node's shell (termproxy/vncshell) for direct root on the host
- read cluster config, users, tokens, and storage definitions
Permissions are path-scoped, so a limited token still enables whatever its role allows.
```

## Exploitation notes

- API tokens are bearer credentials scoped by role; a found token (in scripts, CI, or config) acts with its permissions directly, no password needed, so hunt for `PVEAPIToken` strings.
- The guest-agent exec endpoint runs commands inside a VM from the management plane, and the termproxy/vncshell endpoints open a node root shell, so management access converts to host and guest execution.
- Enumerate the role grants: a non-root user or token with VM or node privileges may still allow console access, backup restore, or disk attach that reach data; `access/permissions` shows the scope.
- Fingerprint the Proxmox version for any ticket/token or privilege-escalation advisory in the management stack.

## References

- [Proxmox VE API viewer](https://pve.proxmox.com/pve-docs/api-viewer/)
- [Proxmox VE: user management and permissions](https://pve.proxmox.com/pve-docs/chapter-pveum.html)
