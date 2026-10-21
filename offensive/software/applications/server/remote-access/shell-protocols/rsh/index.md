---
title: "rsh: attacking the remote shell service"
order: 5
description: "rsh (rshd on TCP 514) executes a command or opens a shell on a remote host. Traditional rshd requires a privileged client source port, validates host and user identity, and consults .rhosts or /etc/hosts.equiv rather than requesting a password. Attacks abuse that trust or intercept the cleartext exchange."
keywords:
  - rsh
  - rshd
  - port 514
  - rhosts
  - remote shell
---

# rsh

`rsh` runs a command (or an interactive shell) on a remote host via `rshd` on TCP 514. Traditional implementations first require a client source port in the privileged 512–1023 range, may verify that reverse and forward hostname resolution agree, retrieve the remote username, requested local account, and command, and then evaluate `~/.rhosts` or `/etc/hosts.equiv` through `ruserok`. A matching trust rule grants access without a password. An attacker who satisfies that complete protocol and trust decision—by controlling a trusted identity, planting an acceptable `.rhosts`, abusing a permissive `hosts.equiv`, or performing a protocol-correct spoofing attack—can run commands as the target user. The resulting streams are unencrypted and therefore interceptable.

```bash
# if trusted (or trust is permissive), run commands with no password
rsh -l <user> <target> id
rsh -l root <target> 'cat /etc/shadow'
# test reachability
nmap -p514 -sV <target>
```

## Subtopics

- **[rhosts bypass](rhosts-bypass.md)**: passwordless access via .rhosts trust.
- **[hosts.equiv](hosts-equiv.md)**: system-wide trust abuse.
- **[IP spoofing](ip-spoofing.md)**: impersonating a trusted source address.
- **[Cleartext interception](cleartext-interception.md)**: capturing the unencrypted session.

## References

- [rshd manual](https://linux.die.net/man/8/rshd)
- [HackTricks: rsh (514)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
