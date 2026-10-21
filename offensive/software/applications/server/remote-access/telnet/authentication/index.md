---
title: "Authentication: attacking Telnet login"
order: 1
description: "Telnet commonly authenticates with a username and password over a plaintext session unless an encryption extension is negotiated. Attacks include vendor defaults, services with no authentication, and online password guessing."
keywords:
  - telnet authentication
  - default credentials
  - no authentication
  - brute force
  - cleartext
---

# Authentication

Telnet commonly carries its username/password exchange in plaintext unless an encryption extension is successfully negotiated. Authentication fails in three common ways: a device retains vendor defaults, a service requires no authentication and drops directly to a shell or menu, or an online password attack succeeds against weak credentials. A valid login yields a terminal whose privileges depend on the particular device and account.

```bash
nc <target> 23        # try default/no-auth first
hydra -L users.txt -P passwords.txt telnet://<target> -t 4
```

## Subtopics

- **[Default credentials](default-credentials.md)**: vendor and factory Telnet logins.
- **[No authentication](no-authentication.md)**: servers that drop to a shell without a password.
- **[Password brute force](password-brute-force.md)**: online attacks against weak login credentials.

## References

- [HackTricks: Telnet authentication](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
