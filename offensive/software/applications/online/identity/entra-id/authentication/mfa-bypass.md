---
title: "MFA bypass: defeating or enrolling second factors"
order: 6
description: "Defeating Entra multi-factor authentication: MFA fatigue, SIM and token theft, self-service registration of attacker factors, and authentication-method manipulation."
keywords:
  - MFA bypass
  - MFA fatigue
  - authentication methods
  - SSPR
  - token theft
---

# MFA bypass

Once a password is valid, MFA is the next gate, and it falls in several ways: wear the user down with repeated push prompts, steal a token that already satisfied MFA, or register your own factor on an account that has none yet.

## Register an attacker factor

```bash
# when an account has no MFA method registered, enrol your own at first sign-in
# via https://aka.ms/mfasetup or the authentication-methods Graph endpoint
# (self-service registration with no existing factor is the classic takeover)
```

## Manipulate methods with Graph

```bash
# with the right role, add a phone/authenticator method to a victim and own their MFA
# POST /users/{id}/authentication/phoneMethods  (Graph)
```

## Exploitation notes

- **MFA fatigue**: repeated push notifications until the user approves; works where number-matching is not enforced.
- A stolen **PRT** or **refresh token** already embodies MFA, so token replay sidesteps the prompt entirely ([token theft](token-theft.md)).
- An account enrolled by the attacker (no prior factor) is a full takeover; sweep spray hits for MFA-not-registered state.

## Tools

- **TeamFiltration** / **TokenTactics**: capture tokens that carry the MFA claim.
- **Graph / AADInternals**: register or read authentication methods.

## References

- [HackTricks Cloud: MFA bypass](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [AADInternals: MFA](https://aadinternals.com/aadinternals/)
- [dirkjanm.io](https://dirkjanm.io/)
