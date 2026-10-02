---
title: "Server-side markup injection: XXE, entity-expansion DoS, XPath, and stored HTML"
description: How attacker-controlled XML, XPath, and HTML reach a server-side interpreter, external entities for file read and SSRF, entity expansion for denial of service, predicate injection in XPath queries, and sanitizer gaps in stored rich text.
keywords:
  - markup injection
  - XXE
  - XML external entity
  - XML bomb
  - XPath injection
  - HTML sanitization
  - stored XSS
---

# Markup injection

**Markup injection** covers the vulnerabilities that arise when a server treats attacker-supplied text as *structured markup*, XML, XPath expressions, or HTML, and the interpreter honors features the application never meant to expose. The primary sink is on the server: an XML parser that resolves entities, a query engine that evaluates an XPath string, or an HTML cleaner whose allowlist leaks. The payoffs range from local file disclosure and server-side request forgery, through denial of service, to authentication bypass and stored script execution.

## Overview

Applications parse structured documents everywhere: SOAP and XML-RPC endpoints, SAML assertions, SVG and DOCX uploads, RSS and sitemap ingestion, configuration files, and rich-text comment fields. Each format is handled by a parser that carries decades of optional, powerful features, external entity resolution, DTD processing, document inclusion, most of which are irrelevant to the application but enabled by default in older stacks.

The unifying theme across these pages is the same one that defines every injection class: a value crosses from **data** into **code**. The attacker's bytes are not merely parsed as content; they are interpreted as *instructions* by the markup engine.

- **XML parsers** can be steered into reading files and making outbound requests through external entities, or into exhausting memory through recursive entity expansion.
- **XPath engines** evaluate query strings built by concatenation, so an injected quote and `or` clause rewrites the predicate, exactly as classic SQL injection rewrites a `WHERE` clause.
- **HTML sanitizers** try to render untrusted markup safely, but a mismatch between how the cleaner parses a document and how a browser later parses it re-opens cross-site scripting.

A fetch triggered *only* as a side effect of XML parsing (an external entity pointing at an internal URL) overlaps with the fetch primitive described under [Request forgery](../request-forgery/index.md); here the focus stays on the markup interpreter as the entry point.

## The surface

The decisive questions when assessing any markup sink:

- **Who parses first?** If the server's XML/HTML/XPath engine is the primary interpreter, the bug lives here; if the browser is the real sink, it is a client-side XSS concern.
- **Which features are on?** DTDs, external and parameter entities, and entity expansion are the high-value XML toggles. For XPath, it is the lack of parameterization. For HTML, it is the exact allowlist and the parser's handling of foreign content (SVG, MathML).
- **Is output reflected?** Many XML sinks are blind, no parsed result returns to the page, so exploitation shifts to out-of-band (OOB) channels, error-based leakage, and timing, mirroring blind SQL injection.

## Pages

| Page | Focus |
|------|--------|
| [XML External Entity (XXE)](xml-external-entity-xxe.md) | External and parameter entities for local file read, SSRF, and OOB/blind exfiltration via DTDs |
| [XML entity expansion DoS](xml-entity-expansion-dos.md) | Billion laughs and quadratic blowup, memory and CPU exhaustion through entity amplification |
| [XPath injection](xpath-injection.md) | Predicate concatenation, authentication bypass, and boolean/blind node extraction |
| [Stored HTML and sanitizer gaps](stored-html-and-sanitizer-gaps.md) | Allowlist cleaners, parser differentials, and mutation XSS on server-sanitized rich text |

## References

- [OWASP: XML External Entity (XXE) Processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [CWE-611: Improper Restriction of XML External Entity Reference](https://cwe.mitre.org/data/definitions/611.html)
- [CWE-643: Improper Neutralization of Data within XPath Expressions](https://cwe.mitre.org/data/definitions/643.html)
- [CWE-776: Improper Restriction of Recursive Entity References (XML Bomb)](https://cwe.mitre.org/data/definitions/776.html)
- [CWE-79: Improper Neutralization of Input During Web Page Generation (XSS)](https://cwe.mitre.org/data/definitions/79.html)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
