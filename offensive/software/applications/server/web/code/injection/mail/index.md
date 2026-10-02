---
title: "Mail injection"
description: "Application-layer email abuse where untrusted input reaches message headers, the body, or the SMTP hand-off and is interpreted as part of the message rather than treated as data."
keywords:
  - mail injection
  - email header injection
  - CRLF injection
  - SMTP injection
  - MIME injection
---

# Mail

Mail injection covers the application-layer abuse of email composition: an attacker supplies input that an application concatenates into a message and that the mail stack then interprets as structure rather than content.

The shared primitive is a trust boundary that a web form or API crosses when it builds a message from user data. Contact forms, "invite a friend" features, password-reset mailers, support ticketing, and newsletter signups all take fields such as sender name, subject, and recipient and hand them to a `mail()` call, a mailer library, or a raw SMTP conversation. When those fields pass through unseparated, the newline and the protocol tokens that delimit headers, body, MIME parts, and SMTP commands become attacker-controlled. The subtree splits by where the interpretation happens: header injection at the header block, MIME multipart abuse at the message-structure layer, and SMTP segment injection at the protocol hand-off.

## Tools

- **swaks**: Swiss Army Knife for SMTP, scripting crafted mail envelopes.
- **Burp Repeater**: injecting CRLF sequences into mail form fields.
- **curl**: sending crafted CRLF payloads into mail-composition fields.
- Manual testing with Burp Repeater and crafted payloads.

## References

- [OWASP: Testing for IMAP SMTP Injection (WSTG)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11-Testing_for_HTTP_Splitting_Smuggling)
- [PayloadsAllTheThings: CRLF Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
