---
title: "Enumeration: fingerprinting an SSH service"
order: 3
description: "SSH enumeration gathers what directs every later attack: the version banner that identifies the implementation and its vulnerabilities, the offered algorithms that reveal weak-crypto exposure, the host-key fingerprint, and valid usernames. None requires authentication, and the results select which authentication, crypto, or exploit path applies."
keywords:
  - ssh enumeration
  - banner
  - algorithms
  - user enumeration
  - host key
---

# Enumeration

Before attacking SSH, enumerate it: the version banner names the implementation and build (mapping to known vulnerabilities), the offered key-exchange, cipher, and MAC algorithms reveal weak-crypto exposure, the host-key fingerprint identifies the server (and enables trust attacks), and valid usernames focus credential attacks. All of this is unauthenticated, and each result points at a specific follow-on: a vulnerable version to exploit, weak algorithms to target, or a user list to spray.

```bash
nmap -p22 -sV --script ssh2-enum-algos,ssh-hostkey,ssh-auth-methods <target>
nc <target> 22                                 # raw banner
```

## Subtopics

- **[Banner grabbing](banner-grabbing.md)**: version and implementation disclosure.
- **[Algorithm enumeration](algorithm-enumeration.md)**: offered crypto and weak-option detection.
- **[User enumeration](user-enumeration.md)**: discovering valid usernames.

## References

- [HackTricks: SSH enumeration](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
- [nmap SSH scripts](https://nmap.org/nsedoc/)
