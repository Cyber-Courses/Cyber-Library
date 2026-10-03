---
title: "Exchange enumeration: users and the address list"
description: "Enumerating an on-premises Exchange deployment from outside: fingerprinting the version, validating usernames through OWA/Autodiscover timing, and harvesting the Global Address List through OWA FindPeople or EWS."
keywords:
  - Exchange enumeration
  - Autodiscover
  - OWA
  - global address list
  - MailSniper
---

# Exchange enumeration

Exchange exposes several web endpoints to the internet, and each leaks information before authentication. The goal here is to build the **user list** to spray and the **version** to match against known exploit chains, using only unauthenticated or single-credential access.

## Version and endpoints

OWA and EWS reveal the Exchange build in headers and static resources, which tells you whether a server is in range for a given [RCE chain](rce-chains.md):

```bash
# Build number from response headers, then from the versioned resource path in the
# login HTML: OWA serves its static assets under /owa/auth/<build>/..., which is the build
curl -sk https://<exch>/owa/auth/logon.aspx -I | grep -i 'X-OWA-Version\|X-FEServer'
curl -sk https://<exch>/owa/auth/logon.aspx | grep -oE '/owa/auth/[0-9.]+/' | head -1
```

Key paths: `/owa`, `/ecp`, `/ews`, `/autodiscover`, `/mapi`, `/rpc`, `/powershell`, `/activesync`.

## User validation

Usernames can be validated without a password through response differences and timing on Autodiscover/OWA:

```bash
# Timing-based user enumeration against OWA (valid users respond differently)
# tools: MailSniper Invoke-UsernameHarvestOWA, or a timing script against /autodiscover
```

## Harvesting the Global Address List

Once you have one valid credential, pull the whole organisation's addresses from the **GAL**, turning one account into the full user list:

```powershell
# MailSniper: FindPeople via OWA, falling back to EWS
Get-GlobalAddressList -ExchHostname <exch> -UserName domain\user -Password <pass> -OutFile gal.txt
```

## Exploitation notes

- The GAL gives **real, current** usernames and email formats, far better than a guessed list, so it sharply improves [spraying](password-spraying.md) hit rates.
- Version fingerprinting is the gate to the RCE chains: match the exact build before firing ProxyShell/ProxyNotShell.
- Autodiscover and OWA are reachable even when the rest of the org is firewalled, so this is a true external foothold-builder.

## Tools

- **MailSniper** (`Get-GlobalAddressList`, `Invoke-UsernameHarvestOWA`): GAL and user harvesting from OWA/EWS.
- **ruler** (SensePost): Exchange/MAPI interaction and enumeration.
- **curl / nmap http-exchange scripts**: version and endpoint fingerprinting.

## References

- [MailSniper (dafthack)](https://github.com/dafthack/MailSniper)
- [ruler (SensePost)](https://github.com/sensepost/ruler)
- [Black Hills InfoSec: attacking Exchange with MailSniper](https://www.blackhillsinfosec.com/attacking-exchange-with-mailsniper/)
