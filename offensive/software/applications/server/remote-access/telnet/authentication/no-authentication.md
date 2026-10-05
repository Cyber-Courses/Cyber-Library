---
title: "No authentication: Telnet servers that require no password"
description: "Some Telnet services drop straight to a shell or configuration menu with no login, or accept any username with no password. Common on embedded devices, debug interfaces, and misconfigured equipment, an unauthenticated Telnet grants immediate terminal access, frequently as root or an administrative context, to anyone who connects."
keywords:
  - no authentication
  - open telnet
  - debug interface
  - unauthenticated
  - shell
---

# No authentication

A surprising number of Telnet services require no authentication: embedded devices and debug interfaces drop directly to a shell, some servers accept any username without a password, and misconfigured equipment exposes a configuration menu with no login. Where this is the case, connecting is the attack, and the context is frequently privileged (a root BusyBox shell, a device admin menu, a diagnostic interface). Manufacturer debug/backdoor Telnet interfaces that bypass authentication are a recurring class on consumer and industrial devices.

```bash
nc <target> 23        # connect; a shell/menu with no login prompt => no auth
# some accept any user, empty password:
(printf 'anyuser\n\n'; sleep 1; printf 'id\n') | nc <target> 23
# scan a range for open/no-auth telnet
nmap -p23 --open <range>
```

## Exploitation notes

- The tell is the absence of (or a non-enforcing) login prompt: a direct shell, a menu, or acceptance of any username with an empty password.
- Embedded debug and manufacturer backdoor interfaces are the common source; these often sit on non-standard ports too, so scan beyond 23 on devices.
- The access is typically privileged (root/admin on the device), so an unauthenticated Telnet is usually full device control immediately.
- Where a login is enforced, fall back to [default credentials](default-credentials.md) and [brute force](password-brute-force.md).

## References

- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
