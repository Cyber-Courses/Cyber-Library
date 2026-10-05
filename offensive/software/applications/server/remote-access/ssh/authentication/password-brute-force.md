---
title: "Password brute force: online password attacks against SSH"
description: "Where SSH permits password authentication, it is a brute-force and password-spray target. Success gives a shell as the account, frequently reusable elsewhere. SSH rate limiting, per-connection auth-try limits, and key-only configurations shape the attack, favouring slow spraying of validated usernames with context-derived passwords over fast brute force."
keywords:
  - ssh brute force
  - password spray
  - hydra
  - maxauthtries
  - credential reuse
---

# Password brute force

When `PasswordAuthentication` is enabled, SSH logins can be guessed online. The practical attack is shaped by SSH's defenses: `MaxAuthTries` limits attempts per connection, `MaxStartups` throttles concurrent handshakes, and fail2ban-style tools block offending IPs, so a fast, broad brute force is noisy and self-defeating. A slow spray, one or a few likely passwords across a validated user list, is the effective approach, and any valid credential is prime reuse material because SSH accounts are real system accounts.

```bash
# confirm password auth is offered before spraying
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no user@<target> 2>&1 | head -1
# spray one password across users (quiet), stop on success, few threads
hydra -L users.txt -p 'Autumn2025!' ssh://<target> -t 4 -f
nxc ssh <target> -u users.txt -p 'Autumn2025!'          # marks shell vs no-shell on hit
# brute one high-value account with a short, targeted list
hydra -l admin -P top-passwords.txt ssh://<target> -t 4 -f
```

## Exploitation notes

- Check `PasswordAuthentication` first; many servers are key-only (password auth disabled), in which case pivot to the key routes rather than guessing.
- Keep threads low and the password list short and contextual: SSH logs every failure, `MaxAuthTries` closes the connection after a few tries, and IP-blocking tools react to bursts.
- Validate usernames first ([User enumeration](../enumeration/user-enumeration.md)) so attempts land on real accounts; derive passwords from the organisation (season/year, company name, breached creds) rather than a generic giant wordlist.
- Any working SSH credential is likely reused; test it against other hosts and services immediately. NetExec marks whether the account gets a shell or is confined.

## Tools

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [NetExec (ssh)](https://github.com/Pennyw0rth/NetExec)

## References

- [OpenSSH sshd_config: MaxAuthTries, PasswordAuthentication](https://man.openbsd.org/sshd_config)
- [HackTricks: SSH brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
