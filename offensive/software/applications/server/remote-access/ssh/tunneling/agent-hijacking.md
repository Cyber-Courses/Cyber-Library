---
title: "Agent hijacking: reusing a forwarded SSH agent"
order: 4
description: "SSH agent forwarding exposes a user's agent socket on the remote host so they can authenticate onward without copying keys. An attacker who controls, or has root on, a host where a user forwarded their agent uses that live socket to authenticate as the user to any system their keys unlock, without ever possessing the private key."
keywords:
  - agent forwarding
  - ssh-agent
  - SSH_AUTH_SOCK
  - agent hijacking
  - key reuse
---

# Agent hijacking

SSH agent forwarding (`ssh -A`) lets a user carry their authentication to a remote host: the agent, holding their private keys, stays on their workstation, and a socket on the remote host proxies signing requests back to it. This avoids copying keys to intermediate hosts, but it means that while the user is connected, any process on the remote host that can reach the forwarded agent socket can ask it to sign authentications, as the user, to any system their keys unlock. An attacker with control of (or root on) such a host hijacks the live socket and moves onward as the user, never seeing the private key itself.

```bash
# on a host where a user has forwarded their agent, find the agent socket
ls -l /tmp/ssh-*/agent.*                        # forwarded agent sockets
# as root (or the user), point SSH at the socket and use the user's keys
SSH_AUTH_SOCK=/tmp/ssh-XXXX/agent.1234 ssh-add -l    # list the keys the agent holds
SSH_AUTH_SOCK=/tmp/ssh-XXXX/agent.1234 ssh user@next-internal-host
```

## Exploitation notes

- The window is while the user's session is live: the forwarded socket only works as long as the agent on their workstation is reachable, so hijacking is opportunistic and timed to active sessions.
- Root on the intermediate host can access any user's forwarded agent socket under `/tmp/ssh-*`; a non-root attacker needs access to the specific socket (same UID or a permissions slip).
- You never obtain the private key, only the ability to authenticate while the agent is reachable; use it immediately to reach the next hop or to plant durable access (an `authorized_keys` entry) as the user.
- This is why forwarding an agent to an untrusted or multi-tenant host is dangerous, and why bastions ([Jump host abuse](jump-host-abuse.md)) that receive forwarded agents are prime hijack points.

## Tools

- [ssh-add / ssh-agent](https://man.openbsd.org/ssh-agent)

## References

- [OpenSSH agent forwarding](https://man.openbsd.org/ssh#A)
- [HackTricks: SSH agent hijacking](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
