---
title: "Microsoft Exchange: attacking the on-premises mail server"
description: "The on-premises Microsoft Exchange attack surface, organized perimeter to Active Directory: reading the server build from OWA and ECP to pick a chain, user enumeration and spraying, the front-end-to-back-end RCE chains, PrivExchange coercion to AD, mailbox access, client abuse, and mail-flow persistence."
keywords:
  - Exchange
  - OWA
  - ECP
  - ProxyShell
  - PrivExchange
  - X-OWA-Version
---

# Microsoft Exchange

On-premises Microsoft Exchange sits on the seam between the perimeter and Active Directory. The Client Access (front-end) services are internet-exposed over HTTPS (`/owa`, `/ecp`, `/ews`, `/Microsoft-Server-ActiveSync`, `/autodiscover`, `/mapi`, `/rpc`, `/powershell`), every one of them authenticates against the domain, and the Exchange servers and their security groups hold **high privilege in AD by design**, historically write access over the domain object itself. That combination is why Exchange is one of the most productive targets in a Windows estate: the front end reaches a privileged back end, a foothold on the box runs as `SYSTEM`, and even with no code execution the server can be coerced into authenticating to you. The mailboxes are a trove on their own.

Everything on this surface keys off one fact you establish first: **the exact build number**. The pre-auth and single-credential RCE chains are tightly version bound, and firing the wrong one against a patched build only burns the engagement.

## Triage: read the build, then pick the chain

The build number is leaked unauthenticated in several places. OWA serves its static assets from a versioned path, and the front-end server stamps its version in response headers:

```bash
# Header stamp (X-OWA-Version on newer CUs, X-FEServer names the front-end host)
curl -sk -I 'https://mail.example.com/owa/auth/logon.aspx' | grep -iE 'X-OWA-Version|X-FEServer'

# Versioned static path in the logon HTML: /owa/auth/15.2.1544.4/... IS the build
curl -sk 'https://mail.example.com/owa/auth/logon.aspx' | grep -oE '/owa/auth/[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/' | head -1

# ECP login page carries the same versioned path and confirms /ecp is reachable
curl -sk 'https://mail.example.com/ecp/' | grep -oE '/ecp/[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/' | head -1
```

The leading numbers name the product family, and the full build decides which chain is in range:

- **15.0.x** = Exchange 2013, **15.1.x** = Exchange 2016, **15.2.x** = Exchange 2019. A bare `15.0`/`15.1`/`15.2` without the full dotted build usually means a very old CU that never stripped the path.
- A build **at or below the early-2021 security rollup** is in range for **[ProxyLogon](rce-chains/proxylogon.md)** (pre-auth SSRF to web shell, `SYSTEM`, no credential).
- A build **below the mid-2021 rollup** is in range for **[ProxyShell](rce-chains/proxyshell.md)** (pre-auth Autodiscover path confusion to the PowerShell back end, no credential).
- A build **below the late-2022 rollup** is in range for **[ProxyNotShell](rce-chains/proxynotshell.md)** (needs one valid mailbox credential); if the URL-rewrite mitigation was applied but the box was never fully patched, the **OWASSRF** variant in the same page still lands.
- A reachable `/ecp` with a known or leaked `machineKey` is in range for **[ViewState deserialization](rce-chains/viewstate-deserialization.md)** regardless of family, once you hold a valid ECP session or the static keys.

Match the build first, then read the chain pages below. Where no chain fits, the surface is still fully attackable from the perimeter through enumeration, spraying, and coercion.

## The attack arc

1. **Enumerate** the build, valid users, and the address book from outside.
2. **Spray** the web endpoints to turn a user list into a domain credential.
3. **Execute** a build-matched RCE chain for `SYSTEM`, or **coerce** the server with PrivExchange and relay it into AD when no chain fits.
4. **Read mail** at scale through impersonation, and **abuse Outlook clients** when you hold a mailbox but not the server.
5. **Persist** in mail flow with transport and forwarding rules that survive password resets.

## Pages and subtopics

- **[Enumeration](enumeration.md)**: Autodiscover domain and user discovery, OWA/EWS timing-based username validation, the GAL/OAB dump, the ECP version leak, and the NTLM type-2 internal-name leak from `/rpc` and `/ews`.
- **[Password spraying](password-spraying.md)**: OWA, EWS, Autodiscover, and ActiveSync spraying, lockout-aware cadence, and the endpoints that often skip lockout tracking.
- **[RCE chains](rce-chains/index.md)**: the front-end proxy to privileged back-end pattern, with ProxyLogon, ProxyShell, ProxyNotShell/OWASSRF, and ViewState deserialization.
- **[PrivExchange](privexchange.md)**: coercing the Exchange machine account to authenticate and relaying it to LDAP for DCSync.
- **[Mailbox access](mailbox-access.md)**: EWS impersonation, Exchange Management Shell mailbox export, and org-wide mail search after a credential or the server.
- **[Outlook client abuse](outlook-client-abuse.md)**: rules, forms, and folder home pages that run code on the victim workstation over MAPI/EWS, no server RCE needed.
- **[Mail flow persistence](mail-flow-persistence.md)**: transport rules, journaling, and forwarding that silently copy org mail to an external address.

## References

- [MailSniper (dafthack): Exchange enumeration, spraying, and mailbox search](https://github.com/dafthack/MailSniper)
- [Microsoft: Exchange Server build numbers and release dates](https://learn.microsoft.com/en-us/exchange/new-features/build-numbers-and-release-dates)
- [Black Hills InfoSec: attacking Exchange with MailSniper](https://www.blackhillsinfosec.com/attacking-exchange-with-mailsniper/)
- [Google Cloud / Mandiant: ProxyShell exploiting Microsoft Exchange servers](https://cloud.google.com/blog/topics/threat-intelligence/pst-want-shell-proxyshell-exploiting-microsoft-exchange-servers)
