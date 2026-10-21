---
title: "rexec: attacking the remote execution service"
order: 3
description: "rexec (rexecd on TCP 512) executes a command on a remote host using a username and password sent in cleartext. Attacks include capturing those credentials and abusing unsafe command construction; .rhosts and hosts.equiv belong to rsh/rlogin, not rexec authentication."
keywords:
  - rexec
  - rexecd
  - port 512
  - cleartext
  - remote execution
---

# rexec

`rexec` executes a command on a remote host via `rexecd` on TCP 512. It authenticates with a username and password sent in cleartext, so a positioned attacker can capture reusable credentials. Where a server or wrapper constructs the executed command unsafely from attacker-influenced input, command injection can extend what runs. A successful request executes as the authenticated user. Unlike `rsh` and `rlogin`, `rexec` does not authenticate through `~/.rhosts` or `/etc/hosts.equiv`.

```bash
# run a command and let the client obtain credentials interactively or from its configured source
# the password is still sent unencrypted; passing it with -p may also expose it in the local process list
rexec -l <user> <target> id
nmap -p512 -sV <target>
```

## Subtopics

- **[Cleartext passwords](cleartext-passwords.md)**: capturing the rexec credential.
- **[Trusted-host distinction](trusted-host-bypass.md)**: why rsh/rlogin host trust is a separate service path, not a rexec bypass.
- **[Command injection](command-injection.md)**: abusing unsafe command construction.

## References

- [rexecd manual](https://linux.die.net/man/8/rexecd)
- [HackTricks: rexec (512)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
