---
title: "Markup injection"
order: 5
description: "Injection into server-side markup and XML processing languages: Server-Side Includes, XML parsing, XPath/XQuery/XSLT queries, and XXE entity abuse."
keywords:
  - markup injection
  - SSI injection
  - XML injection
  - XPath injection
  - XXE
---

# Markup

Markup processing languages sit between the request and the response, and many of them execute directives that live inside otherwise-inert text. When user input reaches a document that the server parses as markup rather than treating as data, the parser acts on attacker-supplied directives. This subtree groups the injection families that share that root cause.

**Server-Side Includes (SSI)** let a web server expand `<!--#...-->` directives inside a page before sending it, so injected directives can run commands, echo server variables, and pull in files. **XML processing** covers the parse, query, and transform stages of an XML pipeline: a parser that resolves external entities exposes XXE, while a string-built query exposes injection into **XPath**, **XQuery**, and **XSLT**. In each case the primitive is the same, a boundary between data and instruction that the input crosses, and the payloads differ only in the directive syntax each processor understands.

This section covers SSI directive injection and the XML processing family, including XPath query injection and the entity-level attacks that follow from permissive parsers.

## Subtopics

- **[SSI](ssi/index.md)**: Server-Side Includes expand <!--#...--> directives before a page is served; user input that reaches an SSI-parsed page runs directives for command execution,...
- **[XML processing](xml-processing/index.md)**: Injection across the XML pipeline: parsing, querying with XPath/XQuery, and transforming with XSLT, plus the external-entity abuse that permissive parsers al...

## Tools

- **Burp Suite**: injecting and iterating SSI and XML payloads through Repeater and Intruder.
- **XXEinjector**: automating XXE exploitation under the XML processing branch.
- **xcat**: automating XPath data extraction.
- **Burp Collaborator**: confirming blind and out-of-band injection.

## References

- OWASP: Server-Side Includes (SSI) Injection
- PortSwigger Web Security Academy: XML external entity (XXE) injection
