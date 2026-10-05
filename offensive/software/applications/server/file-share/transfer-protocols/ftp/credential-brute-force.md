---
title: "Credential brute force: attacking the FTP plaintext login"
description: "FTP authentication is plaintext username and password with no built-in rate limiting in many servers, so it is a straightforward brute-force and password-spray target. Valid credentials give access to that user's files and, where the FTP user maps to a system account, often a foothold reusable over SSH or other services."
keywords:
  - ftp brute force
  - password spray
  - hydra
  - cleartext
  - credential reuse
---

# Credential brute force

FTP logins are plaintext user/password pairs, and many FTP servers apply no lockout or rate limiting, which makes online brute force and password spraying practical. Valid credentials give access to that account's files; crucially, FTP users are frequently mapped to real system accounts, so a working FTP credential is often reusable for SSH, SMB, or the OS login, turning file access into a system foothold.

```bash
# spray a known user list with common passwords (quieter), or brute one account
hydra -L users.txt -p 'Winter2025!' ftp://<target>        # spray one password
hydra -l admin -P rockyou.txt ftp://<target> -t 4 -f      # brute one user, stop on hit
# netexec supports FTP too
nxc ftp <target> -u users.txt -p passwords.txt
```

## Exploitation notes

- Prefer spraying one common password across many users over hammering one account, both to find weak accounts and to avoid any lockout that does exist; derive the user list from other enumeration (SMB RID cycling, OSINT).
- FTP accounts commonly are system accounts, so test any working credential against SSH and other services immediately; credential reuse is the main payoff.
- Cleartext login also means a sniffing position captures credentials with no brute force at all; on a shared segment, capture beats guessing.
- A found account's file access may itself be the goal (a backup or data FTP), independent of system reuse.

## Tools

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [NetExec](https://github.com/Pennyw0rth/NetExec)

## References

- [HackTricks: FTP brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp)
- [RFC 959: authentication](https://datatracker.ietf.org/doc/html/rfc959)
