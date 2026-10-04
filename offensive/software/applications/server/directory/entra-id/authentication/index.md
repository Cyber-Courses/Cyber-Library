---
title: "Entra authentication"
description: "Breaking into an Entra tenant through sign-in: user and tenant enumeration, password spraying, device-code and token phishing, primary refresh tokens, and conditional-access and MFA bypass."
keywords:
  - Entra authentication
  - password spraying
  - device code
  - PRT
  - conditional access
---

# Authentication

Getting the first token into an Entra tenant is an outside-in problem: learn which users exist and how the tenant authenticates, then obtain a credential or token without tripping lockout or conditional access. Everything here produces an access or refresh token that the rest of the Entra surfaces consume.

## What folds in here

- **[Enumeration](enumeration.md)**: unauthenticated user, tenant, and federation discovery.
- **[Password spraying](password-spraying.md)**: guessing credentials against the sign-in endpoints within Smart Lockout.
- **[Device code phishing](device-code-phishing.md)**: phishing the OAuth device-code flow for tokens.
- **[Primary refresh token](primary-refresh-token.md)**: stealing and replaying the PRT from a joined device.
- **[Conditional access](conditional-access.md)**: slipping past conditional-access policies.
- **[MFA bypass](mfa-bypass.md)**: defeating or enrolling multi-factor methods.
- **[Token theft](token-theft.md)**: harvesting and replaying access and refresh tokens.

## References

- [AADInternals](https://aadinternals.com/aadinternals/)
- [ROADtools (Dirk-jan Mollema)](https://github.com/dirkjanm/ROADtools)
- [HackTricks Cloud: Azure authentication](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
