---
title: "rexec: attacking the remote execution service"
description: "rexec (rexecd on TCP 512) executes a command on a remote host, authenticating with a username and password sent in cleartext, and can also honour host trust. Attacks are capturing the cleartext credential, abusing trusted-host relationships for passwordless execution, and command injection where the executed command is built unsafely."
keywords:
  - rexec
  - rexecd
  - port 512
  - cleartext
  - remote execution
---

# rexec

`rexec` executes a command on a remote host via `rexecd` on TCP 512. Unlike rsh, it was designed around a username/password, sent in cleartext, and some implementations also honour the host-based trust files. The attacks follow from those two facts. The cleartext credential is captured by a positioned attacker. Trusted-host relationships, where honoured, give passwordless command execution just as with rsh. And where the server or a wrapper builds the executed command from attacker-influenced input unsafely, command injection extends what runs. A successful rexec runs a command as the authenticated user.

```bash
# run a command with credentials (sent in cleartext)
rexec -l <user> -p <password> <target> id
nmap -p512 -sV <target>
```

## Subtopics

- **[Cleartext passwords](cleartext-passwords.md)**: capturing the rexec credential.
- **[Trusted host bypass](trusted-host-bypass.md)**: passwordless execution via host trust.
- **[Command injection](command-injection.md)**: abusing unsafe command construction.

## References

- [rexecd manual](https://linux.die.net/man/8/rexecd)
- [HackTricks: rexec (512)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
