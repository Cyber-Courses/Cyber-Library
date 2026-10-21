---
title: "Telnet: attacking a typically plaintext terminal protocol"
order: 5
description: "Telnet on TCP 23 is plaintext by default, so credentials, commands, and output are exposed unless an encryption extension is successfully negotiated. Its surface includes weak authentication, information disclosure, implementation-specific vulnerabilities, and interception of plaintext sessions."
keywords:
  - telnet
  - port 23
  - cleartext
  - telnetd
  - terminal
---

# Telnet

Telnet is a remote-terminal protocol commonly exposed on TCP 23. Its default data stream is plaintext, so the login, password, commands, and output are visible to a positioned observer. Extensions such as the Telnet ENCRYPT option exist, but support and successful negotiation must be verified and the historic algorithms are not a substitute for SSH. Telnet persists on legacy systems, network equipment, and IoT devices, where its attack surface includes weak authentication, verbose identification, implementation-specific vulnerabilities, and interception or live hijacking of sessions that remain plaintext.

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
- [RFC 2946 (Telnet Data Encryption Option)](https://datatracker.ietf.org/doc/html/rfc2946)
- [HackTricks: Telnet (23)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
