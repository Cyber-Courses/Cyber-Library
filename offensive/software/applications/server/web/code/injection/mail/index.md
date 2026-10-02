---
title: "Mail header and body injection in application-layer email assembly"
description: Outbound email built in application code with unsafe header, envelope, or MIME composition from untrusted input.
keywords:
  - email header injection
  - SMTP
  - MIME
---

# Mail injection

**Mail injection** at the application layer is unsafe assembly of RFC 5322 / MIME messages: newline injection in headers, wrong separation of envelope vs body, and multipart boundary confusion. Pure MTA relay policy without an app string sink is infrastructure-focused elsewhere.

## Pages

| Page | Focus |
|------|--------|
| [SMTP header injection](smtp-header-injection-crlf.md) | CRLF and header smuggling |
| [MIME multipart](mime-multipart-and-attachment-handling.md) | Boundaries and attachments |

## See also

- [Injection (parent)](../index.md)
