---
title: "Password brute force: online guessing against the RDP logon"
description: "RDP accepts Windows credentials, so brute force and password spraying target the logon directly. Windows account lockout and the cost of each RDP handshake make low-and-slow spraying of validated users the practical approach; a hit gives an interactive desktop, often administrative, and is usually reusable across the domain."
keywords:
  - rdp brute force
  - password spray
  - account lockout
  - crowbar
  - nxc rdp
---

# Password brute force

RDP authenticates against Windows accounts, so online guessing goes straight at the logon. Two constraints shape it: Windows account lockout policy (a few wrong attempts can lock a domain account, causing noise and denial) and the relative expense of each RDP connection. The effective method is therefore password spraying, one carefully chosen password across many validated usernames, staying under the lockout threshold, rather than deep per-account brute force. A successful guess returns an interactive desktop, commonly with administrative rights, and the credential is typically reusable across the domain.

```bash
# spray one password across users, respecting lockout (single-threaded, paced)
nxc rdp <target> -u users.txt -p 'Spring2025!'         # flags valid creds and admin access
crowbar -b rdp -s <target>/32 -u admin -C passwords.txt -n 1
hydra -L users.txt -p 'Spring2025!' rdp://<target> -t 1
# derive the lockout threshold first (from SMB/LDAP) to spray safely
nxc smb <dc> -u user -p pass --pass-pol 2>/dev/null
```

## Exploitation notes

- Learn the lockout policy before spraying (via SMB/LDAP `--pass-pol`): spray fewer attempts than the threshold per window, and wait out the reset, or risk locking accounts and alerting defenders.
- Spray validated usernames (from [RDP NTLM info](../enumeration/banner-grabbing.md) domain data plus AD enumeration) in `domain\user` or UPN form; a single common password across the org finds the weak accounts.
- NetExec's RDP module flags not just validity but whether the account has admin access on the target, directing you to the highest-value hits.
- A valid RDP credential is interactive access and strong reuse material; test it on other hosts and services, and expect many RDP accounts to be local admins.

## Tools

- [NetExec (rdp)](https://github.com/Pennyw0rth/NetExec)
- [crowbar](https://github.com/galkan/crowbar)

## References

- [HackTricks: RDP brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
- [Microsoft: account lockout policy](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/account-lockout-policy)
