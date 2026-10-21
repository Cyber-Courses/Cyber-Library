---
title: "Trusted-host distinction: rsh trust is not rexec authentication"
order: 3
description: "rexec requires a username and password and does not use .rhosts or hosts.equiv. If the same host separately exposes rsh or rlogin, those services may be vulnerable to host-trust abuse, but that is a distinct service path rather than a rexec authentication bypass."
keywords:
  - trusted host
  - rexec
  - rhosts
  - passwordless
  - command execution
---

# Trusted-host distinction

`rexec` and the trusted r-commands must not be conflated. `rexecd` receives a username and password for authentication; it does not consult `~/.rhosts` or `/etc/hosts.equiv`. Those trust files are used by services such as `rsh` and `rlogin`. A host may expose both service families, but passwordless execution through `rsh` is a separate endpoint and does not bypass `rexec` authentication.

```bash
# identify the services independently
nmap -sV -p512,513,514 <target>  # rexec, rlogin, and rsh use separate ports
# rexec still requires its username and password; avoid placing the password in argv
rexec -l <user> <target> id
# only a separately exposed rsh service can use matching host trust
rsh -l <user> <target> id
```

## Exploitation notes

- Confirm each exposed service rather than assuming that a host running `rexecd` also runs `rshd` or `rlogind`.
- If `rsh` or `rlogin` is independently available, assess its [`.rhosts`](../rsh/rhosts-bypass.md) and [`hosts.equiv`](../trust-abuse/hosts-equiv-trust.md) rules under that service's authentication model.
- For `rexec` itself, the relevant credential risk is [cleartext password capture](cleartext-passwords.md).

## References

- [man 3 rexec](https://man7.org/linux/man-pages/man3/rexec.3.html)
- [Linux rcmd/ruserok authentication routines](https://man7.org/linux/man-pages/man3/rcmd.3.html)
- [Linux-PAM pam_rhosts](https://man7.org/linux/man-pages/man8/pam_rhosts.8.html)
- [HackTricks: rexec](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
