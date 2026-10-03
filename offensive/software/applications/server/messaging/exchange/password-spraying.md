---
title: "Exchange password spraying: guessing credentials at the perimeter"
description: "Spraying passwords against internet-facing Exchange endpoints (OWA, EWS, ActiveSync) to get a first valid domain credential from outside, using the Exchange account lockout behaviour and a harvested user list."
keywords:
  - Exchange password spraying
  - OWA
  - EWS
  - ActiveSync
  - MailSniper
---

# Exchange password spraying

Exchange's web endpoints authenticate against **Active Directory**, so a password that works against OWA is a domain password. Because the endpoints are internet-facing, spraying them is the classic way to turn a harvested [user list](enumeration.md) into a **domain foothold from outside**, no VPN or internal access required.

## Spraying the endpoints

```powershell
# MailSniper: spray OWA or EWS with one password across the user list
Invoke-PasswordSprayOWA -ExchHostname <exch> -UserList users.txt -Password 'Autumn2026!' -OutFile hits.txt
Invoke-PasswordSprayEWS -ExchHostname <exch> -UserList users.txt -Password 'Autumn2026!'
```

```bash
# ActiveSync is another sprayable endpoint and sometimes bypasses OWA lockout tracking
# tools: MailSniper Invoke-PasswordSprayEAS, or a custom O365/EAS sprayer
```

## Staying under lockout

The same rules as domain [password spraying](../../directory/active-directory/authentication/credentials/password-spraying.md) apply, with Exchange specifics:

- Spray **one password per lockout window** across all users; read the domain lockout policy first.
- **EWS and ActiveSync** can increment `badPwdCount` just like OWA, so count attempts across every endpoint you touch, not per endpoint.
- Internet-facing spraying is logged by the server and often by a WAF/IdP, so pace it and prefer a single high-probability password (season-year, company name).

## Exploitation notes

- A single OWA hit is a **domain credential**: it unlocks authenticated AD enumeration, Kerberoasting, and mailbox access in one step.
- OWA/EWS do not enforce MFA in many on-prem deployments even when the org thinks they do (legacy protocols), so legacy endpoints are the softest target.
- Combine with the [GAL harvest](enumeration.md): a complete, correctly-formatted user list dramatically raises spray success.

## Tools

- **MailSniper** (`Invoke-PasswordSprayOWA` / `-EWS` / `Invoke-PasswordSprayEAS`): endpoint spraying.
- **ruler / o365spray / TREVORspray**: alternative sprayers for Exchange and hybrid endpoints.

## References

- [MailSniper (dafthack)](https://github.com/dafthack/MailSniper)
- [pentestlab: Microsoft Exchange password spraying](https://pentestlab.blog/2019/09/05/microsoft-exchange-password-spraying/)
- [TREVORspray (Black Lantern Security)](https://github.com/blacklanternsecurity/TREVORspray)
