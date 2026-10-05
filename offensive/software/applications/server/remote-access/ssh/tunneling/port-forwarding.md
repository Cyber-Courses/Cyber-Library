---
title: "Port forwarding: crossing network boundaries with SSH"
description: "SSH local (-L) and remote (-R) port forwarding tunnel a TCP service through the SSH connection, letting an attacker reach a service on a network they cannot route to, or expose an attacker-side service on the SSH host's network. This bridges firewalled and segmented boundaries using only an SSH foothold."
keywords:
  - port forwarding
  - ssh -L
  - ssh -R
  - firewall bypass
  - pivot
---

# Port forwarding

Port forwarding tunnels a single TCP service through an existing SSH connection, moving it across a boundary the attacker otherwise cannot cross. Local forwarding (`-L`) opens a port on the attacker's side that connects, through the SSH host, to a destination the SSH host can reach, so a database or admin interface on an internal network becomes reachable locally. Remote forwarding (`-R`) does the reverse, opening a port on the SSH host that connects back to an attacker-side service, useful to expose a payload or callback into the target network. Both need only a working SSH session to the pivot.

```bash
# local forward: reach internal:3306 (not routable to you) via the pivot, on your 13306
ssh -L 13306:10.0.5.20:3306 user@pivot
mysql -h 127.0.0.1 -P 13306 -u app -p          # now hits the internal DB

# remote forward: expose your local:8000 as pivot:8000 inside the target network
ssh -R 8000:localhost:8000 user@pivot          # internal hosts can now reach your service

# keep it backgrounded and non-interactive
ssh -fN -L 13306:10.0.5.20:3306 user@pivot
```

## Exploitation notes

- Local forwarding is the common pivot: it brings an internal, non-routable service (database, web admin, SMB) to the attacker's machine through the SSH host, bypassing the firewall that blocks direct access.
- Remote forwarding delivers inward: expose a reverse-shell listener or tool on the target network via the pivot, or receive a callback from a host that can reach the pivot but not the attacker.
- `GatewayPorts` on the SSH server controls whether a remote-forwarded port binds only to localhost or to all interfaces of the pivot; the latter exposes it to the whole internal segment.
- For reaching many hosts/ports rather than one, use dynamic forwarding instead, see [SOCKS proxy](socks-proxy.md); for multi-hop, see [Jump host abuse](jump-host-abuse.md).

## References

- [OpenSSH: -L and -R](https://man.openbsd.org/ssh)
- [HackTricks: tunneling and port forwarding](https://book.hacktricks.xyz/generic-methodologies-and-resources/tunneling-and-port-forwarding)
