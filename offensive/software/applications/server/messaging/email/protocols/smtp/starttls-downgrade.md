---
title: "STARTTLS downgrade: stripping and injecting around opportunistic TLS"
description: "STARTTLS on port 25 is opportunistic, so a positioned attacker can rewrite the EHLO capability list to drop STARTTLS and keep the peer in cleartext where AUTH and MAIL are sniffable. Separately, the pre-TLS command injection class runs bytes buffered before the handshake as commands inside the encrypted session."
keywords:
  - starttls
  - tls downgrade
  - stripping
  - command injection
  - smtp mitm
---

# STARTTLS downgrade

STARTTLS upgrades a plaintext SMTP connection to TLS in place, but on port 25 the upgrade is opportunistic: the client only tries it if the server advertises `250-STARTTLS` in its EHLO reply, and if it is missing the client proceeds in cleartext. That "if advertised" is the weakness. Two distinct attacks live here. The first is stripping: a machine-in-the-middle rewrites the server's EHLO response to delete the STARTTLS capability, so the peer never upgrades and sends AUTH and MAIL in the clear. The second is pre-TLS command injection: some MTAs and TLS-terminating proxies fail to discard the plaintext receive buffer when the handshake begins, so commands an attacker appends after `STARTTLS` (sent before TLS) are parsed as if they arrived inside the now-trusted encrypted session.

## Preconditions

- Stripping needs an on-path position between the two SMTP peers (ARP/DNS redirection on the LAN, a rogue upstream, a controlled relay hop). It only helps where TLS is not mandatory, which is the default for inbound 25.
- Confirm STARTTLS is actually offered and reachable before attacking it:

```bash
# does the server advertise and complete STARTTLS?
openssl s_client -starttls smtp -connect mail.victim.com:25 -crlf
# ... reaches a TLS handshake and cert => STARTTLS works and is a downgrade target
echo | openssl s_client -starttls smtp -connect mail.victim.com:25 2>/dev/null | openssl x509 -noout -subject
```
A completed handshake confirms opportunistic TLS is in play; a plaintext-only server has nothing to strip but is already sniffable.

## Stripping the capability

The MITM forwards the dialogue but edits the single line that offers the upgrade, so the client's "is STARTTLS available?" test fails:

```text
# server actually sends:
250-mail.victim.com
250-STARTTLS            <-- attacker deletes or mangles this line in transit
250 AUTH LOGIN PLAIN
# client sees no STARTTLS, stays in cleartext, and sends:
AUTH PLAIN AGpzbWl0aAB...   <-- credential captured on the wire
```
Mangling (rewriting `250-STARTTLS` to `250-XXXXXXXX`, or the client's `STARTTLS` command to `XXXXXXXX`, so the verb is unrecognized and answered `500`) achieves the same without changing line length. After the strip, read the cleartext `AUTH`/`MAIL FROM` exactly as in [authentication](authentication.md).

## Pre-TLS command injection

Here you do not strip TLS; you smuggle a command across the handshake boundary. Everything after `STARTTLS\r\n` in the same TCP segment is plaintext the server should throw away, but a vulnerable implementation keeps it and executes it post-handshake, inside the authenticated/encrypted context:

```text
EHLO attacker.test\r\n
STARTTLS\r\nRSET\r\n        <-- "RSET" (or MAIL FROM/AUTH) is buffered BEFORE TLS
<TLS handshake begins>
                           <-- after TLS, the buffered RSET runs as a trusted command
```
Substitute a meaningful command for `RSET`: injecting `MAIL FROM`/`RCPT TO` lets an attacker who later presents a client certificate or rides a trusted session prepend a transaction the server attributes to the secured channel. This buffering flaw has affected multiple MTAs and mail proxies.

## Follow-on

- After a strip, you have cleartext credentials and message contents: feed the creds into mailbox access and wider reuse.
- With pre-TLS injection you can inject commands into the session the server considers trusted; keep this conceptually separate from [SMTP smuggling](smtp-smuggling.md), which abuses end-of-data parsing between two servers rather than the TLS boundary on one connection.

## Tools

- [openssl s_client](https://www.openssl.org/docs/man3.0/man1/openssl-s_client.html)
- [Ettercap](https://github.com/Ettercap/ettercap)

## References

- [RFC 3207 (SMTP over TLS, STARTTLS)](https://datatracker.ietf.org/doc/html/rfc3207)
- [RFC 3207 section 4.2 (discard buffered data after STARTTLS)](https://datatracker.ietf.org/doc/html/rfc3207#section-4.2)
- [Nmap smtp-vuln scripts](https://nmap.org/nsedoc/categories/vuln.html)
