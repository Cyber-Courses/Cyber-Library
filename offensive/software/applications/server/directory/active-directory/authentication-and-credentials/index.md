---
title: "Authentication and credentials: obtaining and reusing Active Directory secrets"
description: "The core of Active Directory attacks: obtaining and abusing authentication material in all its forms, passwords and NTLM hashes, Kerberos tickets, and certificates, through dumping, cracking, roasting, relay, and forgery."
keywords:
  - credential access
  - NTLM
  - kerberos
  - AD CS
  - pass the hash
---

# Authentication and credentials

This is the heart of Active Directory attacks. Passwords, NTLM hashes, Kerberos tickets, and certificates are all the same thing seen from different angles: **authentication material** that proves who you are to the domain. An attacker's central loop is to obtain some form of it, convert between forms, and reuse it to authenticate as a more privileged principal. That is why these are treated together rather than split apart: a cracked hash becomes a password, a password becomes a Kerberos ticket, a ticket or certificate becomes code execution, and code execution dumps more hashes.

## The forms of authentication material

- **Secrets** (passwords and their NTLM/Kerberos hashes): obtained by dumping them from a host's memory or databases, cracking them offline, or guessing them through spraying.
- **NTLM authentication**: the challenge-response exchange, which can be captured, relayed, or replayed with just the hash (pass-the-hash).
- **Kerberos tickets**: requested, roasted for their encryption key, forged outright, or passed to impersonate a principal.
- **Certificates**: issued by AD Certificate Services and usable as authentication material (PKINIT), forged or stolen through template misconfigurations.

## How this section is organized

- **[Credentials](credentials/index.md)**: dumping secrets from hosts and the directory, cracking them, and guessing them through spraying.
- **[NTLM](ntlm/index.md)**: capturing, relaying, and replaying NTLM authentication (pass-the-hash).
- **[Kerberos](kerberos/index.md)**: roasting, ticket forgery, delegation abuse, and ticket reuse.
- **[Certificates](certificates/index.md)**: abusing AD Certificate Services for authentication and escalation.

## References

- The Hacker Recipes: Active Directory movement (credentials, NTLM, Kerberos)
- SpecterOps: Certified Pre-Owned (AD CS)
