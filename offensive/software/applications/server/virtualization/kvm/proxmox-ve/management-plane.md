---
title: "Management plane: the Proxmox cluster API and authentication"
description: "Abusing the Proxmox VE management plane: the REST API and pvesh on 8006, its ticket and API-token authentication and the root@pam realm, and the cluster membership that lets control of one node reach the others, to control VMs, containers, and storage across the cluster."
keywords:
  - Proxmox API
  - pvesh
  - API token
  - root@pam
  - cluster
---

# Management plane

Proxmox is driven by a REST API on `8006`, exposed through the web UI, the `pvesh` CLI, and API tokens. Authentication uses realms (`root@pam` is the superuser) and issues tickets, or long-lived API tokens. The API controls every VM, container, and storage target, and because nodes form a cluster, API or node access to one spreads to the others.

```bash
# Ticket auth, then drive the API
curl -sk -d 'username=root@pam&password=<pw>' https://<node>:8006/api2/json/access/ticket
pvesh get /cluster/resources --type vm        # all VMs across the cluster
pvesh create /nodes/<node>/qemu/<vmid>/agent/exec -command 'id'   # run in a guest via agent
```

## Exploitation notes

- `root@pam` or a privileged API token is cluster-wide control; tokens are found in `/etc/pve`, in automation, and in backups.
- The guest agent exec endpoint runs commands inside guests from the API, a management-plane path into VMs without an escape.
- Cluster membership means compromising one node's `/etc/pve` or corosync trust reaches the whole cluster.

## References

- [Proxmox VE API](https://pve.proxmox.com/pve-docs/api-viewer/)
- [Proxmox VE: user management](https://pve.proxmox.com/wiki/User_Management)
