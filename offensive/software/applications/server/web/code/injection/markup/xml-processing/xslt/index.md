---
title: "XSLT injection"
description: "Attacker-influenced stylesheets or transform parameters let an attacker read files, reach internal services, pull remote stylesheets, and run code in the transform engine."
keywords:
  - XSLT injection
  - server-side transform
  - Xalan
  - Saxon
  - libxslt
---

# XSLT

XSLT transforms an XML source into HTML, text, or another XML document using a stylesheet, and server-side engines run that transform in process: **Xalan** and **Saxon** on the JVM, **libxslt** under PHP and Python, and the **.NET** `System.Xml.Xsl` stack. Injection arises when any part of the stylesheet, or a parameter spliced into it, comes from untrusted input, because a stylesheet is itself executable code with access to document resolution, extension functions, and in several engines the host runtime.

The reach of an XSLT injection depends on the processor and its configuration. Every engine resolves URIs through `document()`, giving file read and SSRF. `xsl:import` and `xsl:include` pull in remote stylesheets. Extension functions and embedded `xsl:script` or `msxsl:script` blocks run Java, JScript, or C# directly, escalating a transform to code execution. Engine fingerprinting through `system-property()` picks the right payload.

This subtree covers document-function abuse, extension-function and embedded-script abuse, and remote stylesheet import.
