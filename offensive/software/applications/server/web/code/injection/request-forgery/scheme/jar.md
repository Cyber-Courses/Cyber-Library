---
title: "jar: URLs and Java URLConnection SSRF: nested schemes, jar:http chains, and JDK-specific behavior"
description: Java-specific URL schemes (jar, netdoc) that may chain with other handlers in SSRF research.
keywords:
  - SSRF
  - Java
  - jar URL
---

# jar: (Java SSRF)

Java’s `URL` and `URLConnection` family can resolve `jar:` and nested schemes. SSRF research on JVM apps sometimes chains `jar:http://...!/` style URLs. Behavior is highly version-specific; reproduce only on pinned JDK builds in a lab.
