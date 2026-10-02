---
title: "HTTP request body and content-type confusion"
description: "Abusing how servers parse the HTTP request body: content-type confusion, body-versus-query precedence, and multipart handling to bypass validation and reach unexpected parsers."
keywords:
  - content-type confusion
  - request body parsing
  - multipart
  - mass assignment
  - parameter precedence
---

# Request body

How a server interprets the request body depends on the `Content-Type` it is told, and mismatches between what the client sends, what a filter inspects, and what the framework parses create bypasses.

Content-type confusion sends the body in a format the target does not expect but still parses. A framework that accepts both form-encoded and JSON may apply different binding or validation to each, so switching `Content-Type` can reach a more permissive parser or enable mass assignment of fields the form path would reject:

```
POST /api/profile HTTP/1.1
Content-Type: application/json

{"name":"x","is_admin":true}
```

where the same endpoint, read as a form, would have ignored `is_admin`. A WAF that only inspects form bodies may also miss a payload delivered as JSON or XML.

Body-versus-query precedence is a related bug: many frameworks merge query and body parameters, and when both define the same name, one scope wins. Supplying the allowed value in the inspected scope and the attacker value in the used scope overrides a decision after validation.

Multipart bodies add their own surface: inconsistent boundary parsing, duplicated part names, and `filename`/`Content-Type` fields inside a part can smuggle values past validation or steer file handling. Charset declarations (`charset=`) can change how bytes decode, occasionally reviving filtered characters.

The approach is to resend a request with the body re-encoded under a different content type, with parameters duplicated across scopes, and with multipart parts manipulated, watching for a validation or binding difference. Reach depends on which parsers the framework enables, so confirm what the target accepts.

## References

- OWASP Testing Guide: Testing for Content Type and parameter handling
- RFC 9110: HTTP content and media types
