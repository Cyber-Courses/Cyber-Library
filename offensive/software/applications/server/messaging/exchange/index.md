---
title: "Microsoft Exchange: attacking the on-premises mail server"
description: "The on-premises Microsoft Exchange attack surface: user and address-list enumeration, password spraying against OWA/EWS, the pre-authentication RCE chains (ProxyLogon, ProxyShell, ProxyNotShell), PrivExchange relay to Active Directory, and post-compromise mailbox access."
keywords:
  - Exchange
  - OWA
  - EWS
  - ProxyShell
  - PrivExchange
---

# Microsoft Exchange

On-premises Microsoft Exchange sits at the intersection of the perimeter and Active Directory: it is internet-exposed (OWA, EWS, ActiveSync, Autodiscover), it authenticates every user, and it holds **high privileges in AD by design** (historically write access over the domain object). That combination makes it one of the most productive targets in a Windows estate: a foothold on Exchange, or even just coercing it, often leads straight to Domain Admin, and its mailboxes are a trove on their own.

## Why Exchange matters to an attacker

- It is **externally reachable**, so enumeration and password spraying work from the internet with no prior access.
- It runs **privileged in AD**: the Exchange servers and groups hold powerful rights, so compromising Exchange (or relaying its machine account) reaches the domain.
- It has suffered **pre-authentication RCE** chains (ProxyLogon, ProxyShell, ProxyNotShell) that give SYSTEM on the server directly.
- Its **mailboxes** contain credentials, internal intel, and the ability to send as trusted users.

## Pages

- **[Enumeration](enumeration.md)**: finding users and the Global Address List through Autodiscover, OWA, and EWS.
- **[Password spraying](password-spraying.md)**: guessing credentials against OWA, EWS, and ActiveSync.
- **[RCE chains](rce-chains.md)**: ProxyLogon, ProxyShell, and ProxyNotShell pre-auth to SYSTEM.
- **[PrivExchange](privexchange.md)**: coercing Exchange to authenticate and relaying it to Active Directory.
- **[Mailbox access](mailbox-access.md)**: reading and impersonating mailboxes after compromise.

## References

- [MailSniper (dafthack): Exchange enumeration, spraying, and mailbox search](https://github.com/dafthack/MailSniper)
- [Microsoft: securing on-premises Exchange Server](https://learn.microsoft.com/en-us/exchange/plan-and-deploy/post-installation-tasks/security-best-practices/security-best-practices)
- [Black Hills InfoSec: attacking Exchange with MailSniper](https://www.blackhillsinfosec.com/attacking-exchange-with-mailsniper/)
