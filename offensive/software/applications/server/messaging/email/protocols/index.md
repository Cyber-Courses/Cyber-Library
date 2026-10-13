---
title: "Email protocols: attacking SMTP, IMAP, and POP3 services"
order: 2
description: "Attacking the email wire protocols at the service: SMTP for mail transport and submission, and IMAP and POP3 for mailbox access, across enumeration, relay, authentication, transport security, and server software exploitation."
keywords:
  - SMTP
  - IMAP
  - POP3
  - mail protocol
  - mail server
---

# Protocols

The mail protocols are attacked at the running service. SMTP moves mail between servers (25) and accepts authenticated submission (587, 465); IMAP (143, 993) and POP3 (110, 995) hand a mailbox to a client. Each exposes the same recurring questions: can you enumerate accounts, relay or spray against it, strip its transport security, and is the server software itself exploitable.

## Triage

```bash
nmap -sV -p25,110,143,465,587,993,995 --script \
  smtp-commands,smtp-open-relay,imap-capabilities,pop3-capabilities <target>
# read the banner to identify the daemon, then route to the matching page
printf 'EHLO x\r\nQUIT\r\n' | nc <target> 25     # advertised verbs: AUTH, STARTTLS, VRFY, SIZE
```

A 25/587/465 service routes to [SMTP](smtp/index.md); a 143/993 service to [IMAP](imap/index.md); a 110/995 service to [POP3](pop3/index.md). The banner (Postfix, Exim, Sendmail, Dovecot, Cyrus) also decides the server-exploitation path inside each.

## Subtopics

- **[SMTP](smtp/index.md)**: user enumeration, open relay, authentication, STARTTLS downgrade, SMTP smuggling, and mail-transfer-agent exploitation.
- **[IMAP](imap/index.md)**: enumeration, authentication, mailbox access, and server exploitation.
- **[POP3](pop3/index.md)**: enumeration, authentication, mailbox access, and server exploitation.

## References

- [RFC 5321: SMTP](https://www.rfc-editor.org/rfc/rfc5321)
- [RFC 3501: IMAP4rev1](https://www.rfc-editor.org/rfc/rfc3501)
- [RFC 1939: POP3](https://www.rfc-editor.org/rfc/rfc1939)
