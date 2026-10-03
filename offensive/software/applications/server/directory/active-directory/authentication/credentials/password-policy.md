---
title: "Password policy enumeration: thresholds for safe spraying"
description: "Reading the Active Directory domain password policy and fine-grained password policies to learn lockout thresholds and observation windows before password spraying, avoiding account lockouts."
keywords:
  - password policy
  - lockout threshold
  - fine-grained password policy
  - spraying
  - badPwdCount
---

# Password policy

Before any password guessing, read the lockout policy. Spraying that ignores the lockout threshold locks out accounts, which is noisy, disruptive, and often a scope violation. The policy tells you how many attempts are safe and how long to wait between rounds.

## Reading the default domain policy

The domain policy lives on the domain object and is readable by any authenticated user (and sometimes anonymously):

```bash
nxc smb <dc> -u user -p pass --pass-pol
ldapsearch ... -b 'DC=example,DC=local' -s base \
  lockoutThreshold lockoutDuration lockOutObservationWindow \
  minPwdLength maxPwdAge pwdHistoryLength pwdProperties
```

```
# From a domain host
net accounts /domain
Get-DomainPolicy | Select -Expand SystemAccess
```

The values that govern spraying:

- **`lockoutThreshold`**: bad attempts before lockout (0 means no lockout, spray freely).
- **`lockOutObservationWindow`**: how long before the bad-password counter resets. Spray one attempt per account, then wait slightly longer than this window before the next.
- **`lockoutDuration`**: how long a locked account stays locked.

These are stored as negative 100-nanosecond intervals, so convert them; a `lockOutObservationWindow` of `-18000000000` is 30 minutes.

## Fine-grained password policies

A domain can define **fine-grained password policies** (FGPP, stored as `msDS-PasswordSettings` objects in the Password Settings Container) that override the default for specific users or groups, often with *tighter* lockout for admins or *looser* settings for service accounts. Enumerate them so a per-account policy does not surprise you:

```
Get-DomainObject -SearchBase 'CN=Password Settings Container,CN=System,DC=example,DC=local'
```

## Spraying safely

With the policy known:

- Keep attempts per account per observation window **below** `lockoutThreshold`, typically one attempt per window to be safe.
- Read each account's current `badPwdCount` before spraying so you do not push an account already near the threshold over the edge.
- Prefer a known lockout-safe path where possible: AS-REP roasting and Kerberos pre-auth validation never increment the bad-password count.

## Exploitation notes

- `badPwdCount` is **not replicated** between DCs, so a strict operator sprays through a single DC to keep an accurate per-account count.
- A `lockoutThreshold` of 0 is common in labs and some production domains and means spraying carries no lockout risk at all, only detection risk.
- The policy read is enumeration; the spraying itself is covered under credential bruteforcing in the authentication and credentials section.

## Tools

- **NetExec (nxc) --pass-pol**: default policy read.
- **ldapsearch**: raw policy attributes, including FGPP objects.
- **net accounts / PowerView**: on-host policy read.

## References

- [NetExec: --pass-pol](https://github.com/Pennyw0rth/NetExec)
- [Microsoft: Account Policies (password, lockout, Kerberos)](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/account-policies)
- [enum4linux-ng (cddmp)](https://github.com/cddmp/enum4linux-ng)
