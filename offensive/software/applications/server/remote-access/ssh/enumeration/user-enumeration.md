---
title: "User enumeration: discovering valid SSH usernames"
description: "SSH can reveal which usernames are valid through differences in accepted authentication methods per user, and, on specific OpenSSH versions, a timing side channel where valid users take measurably longer to reject. A validated user list focuses password spraying and key attacks on real accounts and avoids noise against non-existent ones."
keywords:
  - ssh user enumeration
  - auth methods
  - timing attack
  - openssh
  - username
---

# User enumeration

Knowing which usernames exist focuses credential attacks and avoids wasted, noisy attempts against accounts that are not there. SSH leaks this in two ways. First, the authentication methods the server will offer can differ per user, a user configured for keys only versus one allowing passwords, or an existing versus non-existent account, which a probe of accepted methods distinguishes. Second, specific OpenSSH versions had a timing vulnerability: the server performed a real (slow) password hash for valid users but bailed early for invalid ones, so response timing revealed account existence.

```bash
# method-difference probe: compare accepted auth methods across candidate users
for u in root admin deploy svc backup; do
  echo -n "$u: "; ssh -o PreferredAuthentications=none -o StrictHostKeyChecking=no "$u@<target>" 2>&1 \
    | grep -i 'authentications that can continue'; done
# timing-based enumeration on vulnerable OpenSSH versions
nmap -p22 --script ssh-brute --script-args userdb=users.txt <target>   # (careful: attempts auth)
# dedicated tools implement the timing/version-specific checks
```

## Exploitation notes

- The method-difference signal is version-independent and low-noise where it exists: a user offered `publickey` only versus one offered `publickey,password` distinguishes account configuration and sometimes existence.
- The timing side channel applies only to specific OpenSSH versions; match the [banner](banner-grabbing.md) version before relying on it, and account for network jitter by sampling repeatedly.
- Seed candidates from other sources (SMB RID cycling, web app usernames, email/OSINT, service-account conventions) and validate here before [password brute force](../authentication/password-brute-force.md).
- Enumeration that actually attempts authentication is logged; prefer the non-authenticating method probe where the goal is to stay quiet.

## Tools

- [ssh-audit](https://github.com/jtesta/ssh-audit)
- [nmap ssh scripts](https://nmap.org/nsedoc/)

## References

- [OpenSSH user enumeration advisories](https://www.openssh.com/security.html)
- [HackTricks: SSH user enumeration](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
