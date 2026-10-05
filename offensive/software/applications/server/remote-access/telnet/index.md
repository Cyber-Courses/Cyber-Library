---
title: "Telnet: attacking the unencrypted terminal protocol"
description: "Telnet on TCP 23 provides remote terminal access with no encryption, so credentials, commands, and output cross the network in cleartext. Its surface is weak authentication (defaults, no-auth, brute force), a verbose banner that discloses the OS, memory-corruption RCE in old telnetd implementations, and interception of the cleartext session for credentials and hijacking."
keywords:
  - telnet
  - port 23
  - cleartext
  - telnetd
  - terminal
---

# Telnet

Telnet is a remote-terminal protocol on TCP 23 with no encryption whatsoever: the login, the password, every command, and all output travel in cleartext. It persists on legacy systems, network equipment, and IoT devices. The attack surface is accordingly broad: weak authentication (vendor defaults, no authentication, and brute force against the plaintext login), an often-verbose banner that discloses the OS and device, memory-corruption remote code execution in old `telnetd` implementations, and, because the channel is cleartext, straightforward interception of credentials, commands, and even live session hijacking for a positioned attacker.

```bash
nmap -p23 -sV --script telnet-ntlm-info,telnet-encryption <target>
nc <target> 23                                 # banner and login prompt
```

## Subtopics

- **[Enumeration](enumeration/index.md)**: banner and OS detection.
- **[Authentication](authentication/index.md)**: defaults, no-auth, and brute force.
- **[Memory corruption](memory-corruption/index.md)**: RCE in telnetd implementations.
- **[Traffic interception](traffic-interception/index.md)**: cleartext capture and session hijacking.

## References

- [RFC 854 (Telnet protocol)](https://datatracker.ietf.org/doc/html/rfc854)
- [HackTricks: Telnet (23)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
