---
title: "Authentication: attacking Telnet login"
description: "Telnet authenticates in cleartext with a username and password, and is attacked through vendor default credentials (rampant on devices), servers configured with no authentication or a direct shell, and brute force against the plaintext login. A valid login gives a terminal, often on network equipment or an embedded device with administrative scope."
keywords:
  - telnet authentication
  - default credentials
  - no authentication
  - brute force
  - cleartext
---

# Authentication

Telnet's authentication is a cleartext username/password exchange, and it falls three ways. Default credentials are endemic on the devices that still run Telnet, routers, switches, cameras, printers, IoT, so the documented vendor pair often works. Some servers require no authentication, dropping straight to a shell or menu. And the plaintext login is brute-forceable, bounded mainly by the device's (often absent) rate limiting. A successful login yields a terminal, frequently on equipment where that terminal is administrative (an enable-capable router, a root BusyBox shell).

```bash
nc <target> 23        # try default/no-auth first
hydra -L users.txt -P passwords.txt telnet://<target> -t 4
nxc telnet <target> -u users.txt -p passwords.txt 2>/dev/null
```

## Subtopics

- **[Default credentials](default-credentials.md)**: vendor and factory Telnet logins.
- **[No authentication](no-authentication.md)**: servers that drop to a shell without a password.
- **[Password brute force](password-brute-force.md)**: online attacks on the plaintext login.

## References

- [HackTricks: Telnet authentication](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
