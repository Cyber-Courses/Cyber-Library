---
title: "IMAP: attacking stateful mailbox access"
description: "Attacking IMAP, the stateful mailbox-access protocol on 143 (cleartext/STARTTLS) and 993 (implicit TLS). Covers capability and user enumeration, cleartext and SASL authentication, post-auth mailbox reading and search for loot, and server-implementation flaws in Dovecot, Cyrus, and Courier."
keywords:
  - imap
  - 993
  - dovecot
  - mailbox access
  - mail harvesting
---

# IMAP

IMAP keeps mail on the server and lets a client work against it in place: list folders, select a mailbox, search, and fetch individual messages or headers. It listens on 143 (cleartext, often with STARTTLS) and 993 (implicit TLS), and unlike POP3 it is stateful and tagged, so every client command carries a tag the server echoes in its final response. Offensive value runs in two directions: valid credentials turn into a full mailbox read (credentials, password-reset links, internal documents, reply chains for phishing), and the server implementation itself (commonly Dovecot, sometimes Cyrus or Courier) carries its own flaws.

## Find and route the service

```bash
nmap -p143,993 -sV --script imap-capabilities,imap-ntlm-info <target>
# capabilities reveal STARTTLS, LOGINDISABLED, and AUTH= mechanisms
# imap-ntlm-info leaks the AD/NetBIOS/domain/FQDN from an NTLM challenge
openssl s_client -connect <target>:993 -quiet      # implicit-TLS session
a LOGIN user pass                                  # tagged login once connected
```

A tagged response of `a OK` means the command succeeded, `a NO` a clean failure (bad credentials, permission denied), and `a BAD` a protocol/syntax error. On 143, run `A1 CAPABILITY` first: `LOGINDISABLED` means cleartext `LOGIN` is refused until STARTTLS is negotiated, and the `AUTH=` tokens list the SASL mechanisms available for spraying.

## Pages

- **[Enumeration](enumeration.md)**: capability listing, banner-based implementation fingerprint, and username validation through differential responses.
- **[Authentication](authentication.md)**: cleartext `LOGIN`, SASL `AUTHENTICATE` mechanisms, spraying, and what `LOGINDISABLED` changes.
- **[Mailbox access](mailbox-access.md)**: post-auth `LIST`/`SELECT`/`SEARCH`/`FETCH` to read and loot mail, and bulk export.
- **[Server exploitation](server-exploitation.md)**: Dovecot, Cyrus, and Courier implementation flaws and their lineage.

## References

- [RFC 3501: IMAP4rev1](https://datatracker.ietf.org/doc/html/rfc3501)
- [HackTricks: pentesting IMAP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-imap.html)
- [Dovecot documentation](https://doc.dovecot.org/)
