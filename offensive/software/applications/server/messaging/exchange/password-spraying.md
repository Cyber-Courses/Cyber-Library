---
title: "Password spraying: domain credentials at the perimeter"
description: "Spraying passwords against internet-facing Exchange endpoints (OWA, EWS, Autodiscover, ActiveSync) to turn a user list into a domain credential from outside, reading 200 versus 401 on EWS, running lockout-aware cadence, and exploiting the endpoints that skip lockout tracking."
keywords:
  - Exchange password spraying
  - OWA
  - EWS
  - ActiveSync
  - Basic auth
---

# Password spraying

Exchange's web endpoints authenticate against **Active Directory**, so a password that works against OWA or EWS is a domain password. Because the endpoints are internet-facing, spraying them is the standard way to convert a harvested [user list](enumeration.md) into a **domain foothold from outside**, with no VPN or internal access. The mechanics matter: different endpoints enforce lockout differently, and some report a valid credential more cleanly than others.

## Which endpoint, and what a hit looks like

Four endpoints all back onto the same AD authentication, and each is sprayable with Basic or NTLM:

- **OWA** (`/owa/auth.owa`): a form POST; a valid credential redirects into the mailbox, a bad one re-serves the logon form.
- **EWS** (`/ews/exchange.asmx`): HTTP Basic/NTLM; a valid credential returns **200** (or a SOAP fault about the request body), a bad one returns **401**. This is the cleanest signal.
- **Autodiscover** (`/autodiscover/autodiscover.xml`): HTTP Basic/NTLM, same 200-vs-401 behavior.
- **ActiveSync** (`/Microsoft-Server-ActiveSync`): HTTP Basic; frequently **not** tracked by the same lockout counters the org watches on OWA.

A raw EWS probe makes the signal explicit:

```bash
# EWS with Basic auth: 401 = bad credential, 200/500 = the credential authenticated
curl -sk -o /dev/null -w '%{http_code}\n' -u 'EXAMPLE\john.doe:Autumn2026!' \
  'https://mail.example.com/ews/exchange.asmx'
# 200  -> valid, reached the service
# 401  -> rejected credential
# loop a user list at one password, keep every non-401
```

Reading the code is the whole game: `401` is a rejected credential, anything else (200, or a 500 SOAP fault complaining there is no request body) means the credential passed authentication and only the request content was wrong.

## Spray with MailSniper

```powershell
# One password across the whole user list, against OWA then EWS
Invoke-PasswordSprayOWA -ExchHostname mail.example.com -UserList gal.txt -Password 'Spring2026!' -OutFile hits.txt
Invoke-PasswordSprayEWS -ExchHostname mail.example.com -UserList gal.txt -Password 'Spring2026!' -OutFile hits.txt
# ActiveSync, which often bypasses OWA lockout tracking
Invoke-PasswordSprayEAS -ExchHostname mail.example.com -UserList gal.txt -Password 'Spring2026!'
```

`hits.txt` holds `user:password` pairs that authenticated. One line is a domain credential: feed it straight to the [GAL dump](enumeration.md), authenticated AD enumeration, [Kerberoasting](../../directory/active-directory/authentication/kerberos/roasting.md), and [ProxyNotShell](rce-chains/proxynotshell.md), which needs exactly one valid credential.

## Lockout-aware cadence

The same discipline as domain [password spraying](../../directory/active-directory/authentication/credentials/password-spraying.md) applies, with Exchange specifics:

- Spray **one password per lockout window** across all users, never many passwords per user. Read the policy first: authenticated, `Get-ADDefaultDomainPasswordPolicy` or `nxc ... --pass-pol`; unauthenticated, assume the common 5 attempts per 30 minutes and leave margin.
- **Count attempts across every endpoint.** EWS, OWA, and Autodiscover all increment the same `badPwdCount` on the AD account, so three sprays across three endpoints is three failures against the same counter. ActiveSync is the frequent exception that does not feed the counter the org watches, which is why it is the safest spray endpoint and the one to use when the lockout threshold is tight.
- Pace it. Internet-facing spraying is logged by the server and usually by a WAF or identity proxy, so lead with a single high-probability password (season plus year, company name plus year) rather than a dictionary.

## Exploitation notes

- A single OWA/EWS hit is a **domain credential**, not just mail access: it unlocks LDAP enumeration, roasting, and the single-credential RCE path in one step.
- Legacy endpoints (EWS, ActiveSync, Autodiscover with Basic) often escape the MFA the org believes it enforces on OWA, so they are the softest targets even in a nominally MFA-protected estate.
- The [GAL harvest](enumeration.md) and the spray reinforce each other: one credential dumps the real user list, which makes the next password's spray land cleanly instead of locking accounts on invalid names.
- EWS 200-vs-401 is the signal to automate; OWA's form redirect is noisier and slower to parse, so prefer EWS or Autodiscover for the raw spray and keep OWA for confirmation.

## Tools

- **MailSniper** (`Invoke-PasswordSprayOWA` / `-EWS` / `Invoke-PasswordSprayEAS`): endpoint spraying with hit logging.
- **TREVORspray** (Black Lantern Security): distributed, lockout-aware spraying across proxies.
- **o365spray / ruler**: alternative sprayers for Exchange and hybrid endpoints.

## References

- [MailSniper (dafthack)](https://github.com/dafthack/MailSniper)
- [pentestlab: Microsoft Exchange password spraying](https://pentestlab.blog/2019/09/05/microsoft-exchange-password-spraying/)
- [TREVORspray (Black Lantern Security)](https://github.com/blacklanternsecurity/TREVORspray)
