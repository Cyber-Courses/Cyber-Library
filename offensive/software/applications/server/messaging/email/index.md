---
title: "Email: attacking mail protocols, sender authentication, and webmail"
order: 1
description: "The email attack surface: the transport and access protocols (SMTP, IMAP, POP3), the SPF, DKIM, and DMARC sender-authentication records that decide whether a spoof is delivered, and the webmail applications that expose a mailbox over HTTP."
keywords:
  - email
  - SMTP
  - IMAP
  - POP3
  - SPF DKIM DMARC
  - webmail
---

# Email

Email breaks into three attack surfaces that are reached very differently. The wire protocols (SMTP for transport, IMAP and POP3 for access) are attacked at the service: enumeration, relay, authentication, and server software flaws. Sender authentication (SPF, DKIM, DMARC) is attacked at DNS and at the receiver: weak or missing records let an attacker send mail that spoofs the domain and still passes. Webmail is attacked as a web application: the HTTP front end over the mailbox, where XSS, SSRF, and file-write bugs turn into mailbox takeover and code execution.

## Triage

```bash
nmap -sV -p25,110,143,465,587,993,995 <target>     # which mail protocols are live
dig +short TXT <domain> | grep -i spf              # SPF present and how strict
dig +short TXT _dmarc.<domain>                     # DMARC policy: none / quarantine / reject
curl -skI https://<target>/ | grep -i -E 'roundcube|horde|zimbra'   # webmail fingerprint
```

A live SMTP/IMAP/POP3 service routes to [Protocols](protocols/index.md); a permissive or absent SPF/DMARC record routes to [Sender authentication](sender-authentication/index.md); a webmail login page routes to [Webmail](webmail/index.md).

## Subtopics

- **[Protocols](protocols/index.md)**: SMTP, IMAP, and POP3, attacked at the service for enumeration, relay, authentication, and server exploitation.
- **[Sender authentication](sender-authentication/index.md)**: SPF, DKIM, and DMARC record gaps, and the spoofing that abuses them.
- **[Webmail](webmail/index.md)**: Roundcube, Horde, and Zimbra, attacked as web applications in front of the mail store.

## References

- [RFC 5321: SMTP](https://www.rfc-editor.org/rfc/rfc5321)
- [RFC 7489: DMARC](https://www.rfc-editor.org/rfc/rfc7489)
- [HackTricks: pentesting SMTP, IMAP, POP3](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smtp/index.html)
