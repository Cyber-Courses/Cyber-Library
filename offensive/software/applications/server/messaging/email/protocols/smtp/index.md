---
title: "SMTP: attacking the mail transfer service"
description: "SMTP on ports 25, 587, and 465 is the mail transport surface: it leaks valid recipients, relays mail for anyone when misconfigured, authenticates submission clients in cleartext, can be downgraded out of TLS, can be smuggled through to spoof trusted senders, and runs MTA software (Exim, Sendmail, Postfix) with its own exploitation history."
keywords:
  - smtp
  - port 25
  - mail transfer agent
  - open relay
  - smtp enumeration
---

# SMTP

SMTP (Simple Mail Transfer Protocol) moves mail between servers and from clients to servers. It listens on three ports with different roles: TCP 25 for server-to-server (MTA-to-MTA) transfer, TCP 587 for authenticated client submission, and TCP 465 for submission over implicit TLS. The protocol is a plaintext line verb dialogue, so everything it does is directly observable and directly abusable: the verbs that validate recipients leak the user list, a server that forwards mail it should refuse is an open relay, the submission ports carry credentials that may be sniffable, opportunistic STARTTLS can be stripped, and the end-of-data handling can be smuggled through a trusting relay. Underneath the protocol sits an MTA implementation whose banner names it and whose version decides which server-side exploit applies.

## Fingerprint and triage

One scan plus a hand-driven EHLO tell you the MTA, the advertised verbs, and which child applies:

```bash
nmap -p25,465,587 -sV --script smtp-commands,smtp-open-relay,smtp-ntlm-info <target>
# manual: read the banner and the advertised capabilities
nc <target> 25
# 220 mail.victim.com ESMTP Postfix            <- banner names the MTA
EHLO x
# 250-mail.victim.com
# 250-SIZE 52428800
# 250-VRFY                                     <- user-enumeration verb offered
# 250-STARTTLS                                 <- opportunistic TLS, downgrade candidate
# 250-AUTH LOGIN PLAIN                          <- credential surface on this port
# 250 8BITMIME
```

Read the result and route: the banner string (Postfix / Exim / Sendmail / Microsoft ESMTP) points at [server exploitation](server-exploitation/index.md); `VRFY`/`EXPN` or any recipient response difference points at [user enumeration](user-enumeration.md); `AUTH` on 25/587 points at [authentication](authentication.md); `STARTTLS` advertised on 25 points at [STARTTLS downgrade](starttls-downgrade.md); a willingness to accept external MAIL FROM and RCPT TO points at [open relay](open-relay.md). SMTP smuggling is tested against the inbound server regardless of what EHLO advertises.

## Subtopics

- **[User enumeration](user-enumeration.md)**: validate recipients with VRFY, EXPN, and RCPT TO probing.
- **[Open relay](open-relay.md)**: send mail as anyone through a server that relays for external senders.
- **[Authentication](authentication.md)**: AUTH LOGIN/PLAIN credential spraying and cleartext capture on submission.
- **[STARTTLS downgrade](starttls-downgrade.md)**: strip opportunistic TLS and inject pre-handshake commands.
- **[SMTP smuggling](smtp-smuggling.md)**: exploit end-of-data disagreement to inject spoofed, DMARC-passing transactions.
- **[Server exploitation](server-exploitation/index.md)**: MTA-specific attacks against Exim, Sendmail, and Postfix.

## References

- [RFC 5321 (Simple Mail Transfer Protocol)](https://datatracker.ietf.org/doc/html/rfc5321)
- [HackTricks: SMTP (25,465,587)](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smtp/index.html)
- [Nmap NSE: smtp scripts](https://nmap.org/nsedoc/categories/default.html)
