---
title: "Password spraying: guessing valid credentials without lockout"
description: "Guessing Active Directory passwords safely by trying one or a few likely passwords against many accounts, respecting the lockout policy, and using lockout-safe protocols like Kerberos pre-authentication."
keywords:
  - password spraying
  - lockout
  - kerbrute
  - credential stuffing
  - initial access
---

# Password spraying

Spraying inverts the brute-force loop: instead of many passwords against one account (which locks it out), you try **one** likely password against **every** account, then wait and try the next. It is the most common way to get an initial foothold in a domain from an unauthenticated position, and it is safe only if you respect the lockout policy read during [enumeration](../reconnaissance/password-policy.md).

## Choosing candidates

Spraying succeeds on predictable, policy-compliant passwords:

- **Season and year**: `Autumn2026`, `Winter2026!`, matching the complexity rule.
- **Company name plus a suffix**: `Example123`, `Example2026!`.
- **`Password1`, `Welcome1`, `Changeme123`** and default/onboarding passwords.
- **The username as the password**, and empty passwords where `PASSWD_NOTREQD` is set.

Build the account list from [user enumeration](../reconnaissance/user-and-group-enumeration.md).

## Spraying safely

```bash
# Kerberos pre-auth spray: fast and quiet, but a wrong password still counts
# toward lockout, so observe the policy and badPwdCount exactly as with SMB
kerbrute passwordspray -d example.local --dc <dc> users.txt 'Autumn2026!'

# NetExec over SMB/LDAP (watch lockout; use --continue-on-success for a full sweep)
nxc smb <dc> -u users.txt -p 'Autumn2026!' --continue-on-success

# Spray one password, then wait out the lockout observation window before the next
```

The rules that keep accounts unlocked:

- **One attempt per account per observation window**, staying below `lockoutThreshold`.
- **Read `badPwdCount` first** so you do not push an account already near the threshold over.
- **Spray through a single DC**, because `badPwdCount` is not replicated between DCs, so counting across several DCs undercounts and risks lockout.

## Avoiding lockout and guessing altogether

A wrong password increments `badPwdCount` no matter the protocol, Kerberos pre-authentication included, so there is no "free" sprayer that guesses without lockout risk. What is actually free is anything that does not submit a password:

- **Username enumeration** (kerbrute `userenum`): sends an AS-REQ without pre-auth data and reads whether the account exists, so it never touches `badPwdCount`. Use it to trim the list to valid accounts before spraying a single password.
- **AS-REP roasting** sidesteps guessing entirely for accounts that do not require pre-authentication, recovering a crackable hash with no logon attempt (see the Kerberos section).

For the guessing itself, Kerberos pre-auth is fast and quiet, but it is governed by the same lockout policy as SMB: one attempt per account per window, below the threshold.

## Exploitation notes

- A single valid credential, however unprivileged, unlocks full [LDAP enumeration](../reconnaissance/ldap-enumeration.md), [Kerberoasting](../kerberos/spn-discovery.md), and BloodHound collection, so one hit transforms the engagement.
- Spraying is noisy on the authentication logs even when it avoids lockout; pace it and prefer Kerberos to reduce footprint.
- Where a domain sets `lockoutThreshold = 0`, spraying carries no lockout risk at all and can be more aggressive.

## Tools

- **kerbrute**: fast Kerberos pre-auth spraying, plus lockout-free username enumeration (`userenum`) to validate accounts first.
- **NetExec (nxc)**: spraying over SMB, LDAP, WinRM, MSSQL, with lockout awareness.
- **Spray / DomainPasswordSpray**: alternative sprayers that read the policy first.

## References

- The Hacker Recipes: password spraying
- Microsoft: account lockout policy
