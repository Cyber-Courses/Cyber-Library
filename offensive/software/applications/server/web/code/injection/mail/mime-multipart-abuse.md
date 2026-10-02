---
title: "MIME multipart abuse: smuggling parts past a mail composer"
description: "Controlling the MIME boundary or part headers to add attachments, smuggle an alternative body a filter never inspects, or desynchronize how the MTA and the mail client parse the message."
keywords:
  - MIME injection
  - multipart boundary
  - attachment smuggling
  - alternative body
  - Content-Type injection
  - mail parser confusion
---

# MIME multipart abuse

A multipart email is a structured container: a top-level `Content-Type: multipart/...; boundary="X"` header declares a delimiter string, and each part is introduced by `--X` followed by its own headers and content, with `--X--` closing the set. When an application lets untrusted input reach the boundary token or a part's headers, the attacker can forge part separators, add their own parts, or make two parsers disagree about where parts begin and end.

## Where input reaches the structure

Two sinks dominate. First, header injection (CRLF into a header field) that reaches the `Content-Type` line lets the attacker declare or redefine the boundary. Second, templates that interpolate user data into a hand-built multipart body place attacker bytes directly between part separators. In both cases the attacker needs the boundary string, which is often predictable: static in the source, derived from a timestamp, or echoed in an earlier response.

## Adding a part or attachment

Once the boundary is known, injecting a `CRLF` sequence plus a fresh `--boundary` block appends a part the application never intended. Delivering an HTML part:

```
x@evil.test%0d%0aContent-Type: multipart/mixed; boundary="AaB03x"%0d%0a%0d%0a--AaB03x%0d%0aContent-Type: text/html%0d%0a%0d%0a<h1>Phish</h1>%0d%0a--AaB03x--
```

The same shape carries an attachment by setting `Content-Disposition: attachment; filename="invoice.html"`, declaring `Content-Transfer-Encoding: base64`, and a base64 body (the `Content-Transfer-Encoding` header is required, or the client treats the base64 text as literal content and never decodes it into a file). This turns a plain notification mailer into a malware or phishing-document delivery channel sent from the trusted domain.

## Smuggling an alternative body past a filter

`multipart/alternative` tells the client to render the last part it understands, usually the HTML one, while content scanners frequently inspect only the first `text/plain` part. The attacker ships a clean plaintext part and a malicious HTML part under the same boundary:

```
--AaB03x
Content-Type: text/plain

Your account summary is attached.
--AaB03x
Content-Type: text/html

<a href="https://evil.test">Review your account</a>
--AaB03x--
```

The filter clears the innocuous text; the victim's client shows the HTML. The reader and the scanner never see the same message.

## Parser desynchronization

MTAs, gateways, and clients each implement boundary matching slightly differently: handling of leading/trailing whitespace on the delimiter, of a near-miss boundary (`--AaB03x ` with a trailing space), of a duplicate `Content-Type` header, or of a boundary that also appears inside a part's content. Injecting a boundary that one hop treats as a separator and the next treats as literal text makes the gateway and the client parse different structures from the same bytes, so a part visible to one is swallowed by the other. Colliding the declared boundary with a string that appears naturally in the body is a reliable way to split the message at an unexpected point.

## Making it land

- The boundary string must be known or controllable; harvest it from source, leaked headers, or a predictable generator before building parts.
- Part separators and part headers both need real `CRLF`s; on the wire from a web request that is `%0d%0a`, and the blank line before a part's content is a double `%0d%0a%0d%0a`.
- A part must end with `CRLF` immediately before the next `--boundary`, and the final delimiter needs its trailing `--`.

## Tools

- **swaks**: Swiss Army Knife for SMTP, scripting crafted multipart messages.
- **Burp Repeater**: injecting boundary and part-header payloads into mail fields.
- **curl**: sending crafted CRLF and boundary payloads into mail-composition fields.
- Manual testing with Burp Repeater and crafted payloads.

## References

- [OWASP: Testing for IMAP SMTP Injection (WSTG)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_IMAP_SMTP_Injection)
- [PayloadsAllTheThings: CRLF Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
