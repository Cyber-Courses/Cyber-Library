---
title: "Password reset: taking an account over with ForceChangePassword"
description: "Using the User-Force-Change-Password control-access right (or GenericAll) over an Active Directory account to set a new password without knowing the old one, from Linux with net rpc, bloodyAD, and Impacket, and the trade-off against quieter takeovers."
keywords:
  - ForceChangePassword
  - password reset
  - bloodyAD
  - net rpc password
  - account takeover
---

# Password reset

The **`User-Force-Change-Password`** extended right (and `GenericAll`, which includes it) lets you set a **new** password on a target account without supplying the old one, the same operation a helpdesk uses. It is the most direct way to turn a write over a user into control of it: reset the password, then log in as them. The cost is that it is destructive, so it is a deliberate choice rather than the default.

## Resetting from Linux

```bash
# bloodyAD
bloodyAD --host <dc> -d example.local -u user -p pass set password 'victim' 'Newpass123!'

# Samba net (SAMR), works with password or -k for Kerberos
net rpc password 'victim' 'Newpass123!' -U 'example.local/user%pass' -S <dc>

# Impacket (reset via SAMR without knowing the current password)
changepasswd.py example.local/victim@<dc> -reset -altuser example.local/user -altpass 'pass' -newpass 'Newpass123!'
```

From Windows, PowerView does the same:

```powershell
Set-DomainUserPassword -Identity victim -AccountPassword (ConvertTo-SecureString 'Newpass123!' -AsPlainText -Force)
```

## When to use it, and when not to

A reset is loud: the legitimate user can no longer log in, which is noticed quickly and generates account-management events. So:

- **Prefer a non-destructive edge first.** The same `GenericWrite`/`GenericAll` usually also allows [shadow credentials](../authentication/kerberos/shadow-credentials.md) (authenticate as the user without touching the password) or [targeted Kerberoasting](targeted-kerberoasting.md). Use those when the account is in use.
- **Reset is a good fit for** stale or service accounts no one logs into, computer accounts (though resetting a computer password can break the host's domain membership), or when PKINIT is unavailable so shadow credentials are not an option.
- **Restore where possible**: if you captured the old NT hash first, you can set the password back by hash afterwards to reduce disruption.

## Exploitation notes

- `ForceChangePassword` is a *reset* (set new, no old needed), distinct from a *change* (needs the current password); BloodHound's `ForceChangePassword` edge is exactly this right.
- Over a **computer** account, prefer a key-credential or RBCD edge: resetting `MACHINE$` desynchronises the host's secret and is both noisy and disruptive.
- After reset you hold the plaintext, so it feeds straight into authenticated movement and, if the account is privileged, further DACL edges.

## Tools

- **bloodyAD** (`set password`): single-command reset from Linux.
- **Samba `net rpc password` / Impacket `changepasswd.py -reset`**: SAMR-based reset.
- **PowerView** (`Set-DomainUserPassword`): on-host reset.

## References

- The Hacker Recipes: ForceChangePassword
- SpecterOps: BloodHound ForceChangePassword edge
