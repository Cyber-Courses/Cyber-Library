---
title: "XML processing injection"
description: "Injection across the XML pipeline: parsing, querying with XPath/XQuery, and transforming with XSLT, plus the external-entity abuse that permissive parsers allow."
keywords:
  - XML injection
  - XPath injection
  - XQuery injection
  - XSLT injection
  - XXE
---

# XML processing

An XML pipeline moves a document through three stages, and each is a distinct injection surface. The document is **parsed** into a tree, it is **queried** to pull values out, and it may be **transformed** into another format. User input that is concatenated into any of these stages, rather than bound as data, is interpreted as part of the instruction set that stage runs.

At the **parse** stage, a parser configured to resolve external entities turns a crafted document into XXE, reading local files, reaching internal services, and exhausting resources. At the **query** stage, a string-built **XPath** or **XQuery** expression behaves like SQL injection: the input escapes its string literal and rewrites predicates and location paths to select unintended nodes or bypass authentication. At the **transform** stage, **XSLT** processors expose their own functions, and an injected stylesheet fragment can read files, call extension functions, or reach the network.

The shared root cause is that XML blurs the line between data and instruction more readily than most formats, because entities, location paths, and stylesheet directives all travel inside the same document the application treats as input.

This section covers XPath query injection in depth, alongside the wider XML parsing and transformation attacks.
