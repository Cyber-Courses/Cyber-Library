---
title: "Pass-the-ticket: extracting and reusing Kerberos tickets"
order: 3
description: "Stealing Kerberos tickets (TGTs and service tickets) from memory or a ccache and injecting them into another session to authenticate as the victim, including harvesting tickets from a compromised host."
keywords:
  - pass the ticket
  - ccache
  - TGT
  - ticket extraction
  - KRB5CCNAME
---

# Pass-the-ticket

A Kerberos ticket is a bearer token: whoever holds a valid TGT or service ticket can use it, because the ticket itself proves identity to the KDC or service. **Pass-the-ticket** is stealing a ticket that already exists, from memory on a compromised host or from a ccache file, and injecting it into your own session. Unlike overpass-the-hash, it needs no key at all, just the ticket, and it is the natural way to reuse the access a logged-on user already has.

## Harvesting tickets

```bash
# Mimikatz: export all tickets in memory to .kirbi files (needs local admin for others')
sekurlsa::tickets /export
# or the LSA ticket cache
kerberos::list /export

# Rubeus: dump tickets, or monitor for new TGTs as users authenticate
Rubeus.exe dump /nowrap
Rubeus.exe monitor /interval:5
```

On Linux hosts and from Impacket, tickets live in **ccache** files (pointed to by `$KRB5CCNAME`); harvest them from `/tmp`, from compromised service accounts, or convert between `.kirbi` and `.ccache` with `ticketConverter.py`.

## Injecting and using

```bash
# Rubeus: inject a ticket into the current logon session
Rubeus.exe ptt /ticket:ticket.kirbi

# Impacket: point at the ccache and authenticate with -k
export KRB5CCNAME=/tmp/victim.ccache
psexec.py -k -no-pass example.local/victim@<host>
```

## What a stolen ticket gives you

- A stolen **TGT** is the user's full identity: request any service ticket their privileges allow.
- A stolen **service ticket** is narrower: access only to that one service, but enough for a targeted move (for example a CIFS ticket for file access).
- Harvesting from a busy server (an RDP jump host, a Citrix box) collects many users' TGTs, including administrators who logged on there, a classic privilege-escalation harvest.

## Exploitation notes

- Extracting *other* users' tickets from LSASS needs local admin/SYSTEM; your own session's tickets need no special rights.
- Ticket lifetime limits reuse: TGTs default to ~10 hours (renewable ~7 days), so harvested tickets are time-bounded, unlike a hash.
- `Rubeus monitor`/`harvest` captures TGTs as they appear, useful on a host where privileged users log on periodically.

## Tools

- **Rubeus** (`dump`, `monitor`, `ptt`): on-host harvesting and injection.
- **Mimikatz** (`sekurlsa::tickets`, `kerberos::ptt`): memory extraction and injection.
- **Impacket** (`ticketConverter.py`, `-k`): ccache/kirbi handling and use.

## References

- [GhostPack Rubeus (ptt / dump / monitor)](https://github.com/GhostPack/Rubeus)
- [Impacket ticketConverter and -k](https://github.com/fortra/impacket)
- [The Hacker Recipes: pass the ticket](https://www.thehacker.recipes/ad/movement/kerberos/pass-the/ptt)
