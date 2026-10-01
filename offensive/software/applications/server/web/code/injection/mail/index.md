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

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess.

The shared primitive is a trust boundary that a web form or API crosses when it builds a message from user data. Contact forms, "invite a friend" features, password-reset mailers, support ticketing, and newsletter signups all take fields such as sender name, subject, and recipient and hand them to a `mail()` call, a mailer library, or a raw SMTP conversation. When those fields pass through unseparated, the newline and the protocol tokens that delimit headers, body, MIME parts, and SMTP commands become attacker-controlled. The subtree splits by where the interpretation happens: header injection at the header block, MIME multipart abuse at the message-structure layer, and SMTP segment injection at the protocol hand-off.
