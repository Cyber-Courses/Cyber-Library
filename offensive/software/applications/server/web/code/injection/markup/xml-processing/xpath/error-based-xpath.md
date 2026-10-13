---
title: "Error-based XPath extraction"
order: 3
description: "Forcing a cast or type failure in an XPath 2.0 processor embeds node text in the error message, turning a verbose engine into a direct, non-blind extraction channel."
keywords:
  - error-based XPath
  - XPath error message
  - cast failure
  - XPath 2.0 extraction
  - XML error leak
---

# Error-based

When an XPath processor returns its error messages to the client, a deliberately forced failure can carry node text out inside the error string. The reliable form of this is specific to XPath 2.0 and XQuery engines (Saxon, BaseX, eXist-db): their XSD type constructors validate their argument and name the offending value when it fails. XPath 1.0 engines such as libxml2, MSXML, and the native .NET `System.Xml.XPath` engine have no XSD type constructors and do not raise on bad coercions, so against those the error channel confirms and fingerprints injection but does not pull values; the technique below therefore targets a 2.0-capable processor, identified first by the fingerprinting payloads at the end.

## Why a cast leaks data

A cast or type constructor like `xs:integer(...)` parses its argument and, on failure, reports what it could not convert. Feeding node text into a numeric constructor where the text is not a valid number raises a dynamic error whose message quotes the value, for example `FORG0001: Cannot convert string "s3cr3t" to xs:integer`. The password is now in the error.

## Forcing a cast failure

Inject so the surrounding expression stays valid up to the constructor, then push the target node into it:

```
' or xs:integer((//user[1]/pass)) or '
```

When the password is non-numeric, the engine aborts with a `FORG0001`-style message containing the value. Keep each fragment small and unambiguous with `substring`:

```
' or xs:integer(substring((//user[1]/pass),1,20)) or '
```

Advance the offset to read past the first twenty characters, and change the path to move between nodes:

```
' or xs:integer((//user[2]/@role)) or '
```

Where a value happens to be numeric, cast it to a type it cannot satisfy instead, such as `xs:date(...)` or `xs:QName(...)`, so the conversion still fails and reports the text.

## Placing text directly with fn:error

Engines that expose `fn:error()` let an attacker build the fault string, embedding node text with no reliance on a conversion quirk:

```
' or error(xs:QName('x'), string(//user[1]/pass)) or '
```

The processor surfaces the supplied description verbatim, so the node text appears in the error channel directly.

## Leaking structure

Before pulling values, the same mechanism reveals the shape of the document. Forcing `name()` or `local-name()` through a failing cast reports element names one node at a time:

```
' or xs:integer(name(/*[1])) or '
```

The conversion error names the root element; walking indices with `name(/*[1]/*[position()=N])` maps the tree so value extraction knows where to aim.

## Fingerprinting the engine first

Error-based value extraction only works on a 2.0 engine, so confirm the processor before committing to casts. A deliberately malformed expression returns an engine-specific parse error:

```
'
"]
count(//
```

An unbalanced quote or bracket produces a message whose exact wording identifies the engine (libxml2, MSXML, Saxon, .NET). If the fingerprint is a 1.0 engine (libxml2, MSXML, or the native .NET XPath engine), fall back to boolean-blind extraction through the predicate; only a 2.0 engine (Saxon, BaseX, eXist-db) exposes the cast-failure channel above that reads values directly.

## Tools

- **xcat**: automating error-based extraction against XPath 2.0 engines.
- **Burp Repeater**: forcing cast failures and reading leaked node text.
- Manual testing with `xs:integer()` cast and `fn:error()` payloads.

## References

- [OWASP: XPATH Injection](https://owasp.org/www-community/attacks/XPATH_Injection)
- [OWASP WSTG: Testing for XPath Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/09-Testing_for_XPath_Injection)
- [PayloadsAllTheThings: XPATH Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)
