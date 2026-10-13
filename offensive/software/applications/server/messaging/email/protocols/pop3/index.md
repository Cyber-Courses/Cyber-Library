---
title: "POP3: attacking download-only mailbox retrieval"
order: 3
description: "Attacking POP3, the download-and-delete mail retrieval protocol on 110 (cleartext/STARTTLS) and 995 (implicit TLS). Covers banner and CAPA enumeration, USER-based username validation, cleartext USER/PASS and APOP authentication, mailbox download with RETR and TOP, and server flaws in qpopper, Courier, and Dovecot."
keywords:
  - pop3
  - 995
  - apop
  - retr
  - qpopper
---

# POP3

POP3 is the simple cousin of IMAP: it exists to download messages from a single mailbox and, classically, delete them from the server as it goes. It listens on 110 (cleartext, sometimes with STARTTLS) and 995 (implicit TLS). The protocol is a flat set of short commands with `+OK`/`-ERR` replies and no folder model, no server-side search, and minimal state. That simplicity is offensively convenient: a valid credential and a handful of commands dumps the entire mailbox. The same two directions apply as with any mail service, read the mail for loot and reuse the credential, plus the server implementation's own flaws (qpopper on legacy hosts, Courier, Dovecot's POP3 service).

## Find and route the service

```bash
nmap -p110,995 -sV --script pop3-capabilities,pop3-ntlm-info <target>
# pop3-capabilities lists STLS (STARTTLS), SASL, USER, UIDL, APOP
# pop3-ntlm-info leaks domain/host from an NTLM challenge
openssl s_client -connect <target>:995 -quiet     # implicit-TLS session
USER alice
PASS Sprint2025!
```

Every reply is either `+OK` (success, often with data to follow) or `-ERR` (failure with a reason string). Unlike IMAP there are no command tags; responses are positional, so you read them in order. Because POP3 frequently deletes mail on retrieval, prefer `TOP` to peek before you commit to `RETR` on a mailbox you must leave intact.

## Pages

- **[Enumeration](enumeration.md)**: greeting banner, `CAPA`, APOP presence, and `USER`-based username validation.
- **[Authentication](authentication.md)**: cleartext `USER`/`PASS`, the APOP challenge-response, and spraying.
- **[Mailbox access](mailbox-access.md)**: `STAT`/`LIST`/`RETR`/`TOP` to download and loot without destroying the mailbox.
- **[Server exploitation](server-exploitation.md)**: qpopper, Courier, and Dovecot POP3 implementation flaws and their lineage.

## References

- [RFC 1939: Post Office Protocol v3](https://datatracker.ietf.org/doc/html/rfc1939)
- [HackTricks: pentesting POP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-pop.html)
- [nmap pop3-capabilities](https://nmap.org/nsedoc/scripts/pop3-capabilities.html)
