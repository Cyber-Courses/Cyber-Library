---
title: "Password brute force: online guessing against the RDP logon"
description: "RDP accepts Windows credentials, so brute force and password spraying can target the logon. Account-lockout policy and connection cost favor carefully paced testing. A successful login has that account's rights; administrative access and reuse elsewhere are separate findings."
keywords:
  - rdp brute force
  - password spray
  - account lockout
  - crowbar
  - nxc rdp
---

# Password brute force

RDP authenticates against Windows accounts, so online guessing goes straight to the applicable local or domain identity provider. Two constraints shape testing: account-lockout policy and the relative expense of each RDP connection. Password spraying distributes a small number of guesses across accounts, but it does not eliminate lockout or detection risk. A successful guess establishes only that the account can log on to that target; its local privilege and reuse on other systems require separate validation.

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
- Use the resulting session and authorized privilege checks to determine whether the account is a standard user or administrator; do not infer privilege solely from successful RDP authentication.
- Treat reuse on other hosts and services as a separate, lockout-aware test rather than an assumed property of a valid RDP credential.

## Tools

- [NetExec (rdp)](https://github.com/Pennyw0rth/NetExec)
- [crowbar](https://github.com/galkan/crowbar)

## References

- [HackTricks: RDP brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
- [Microsoft: account lockout policy](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/account-lockout-policy)
