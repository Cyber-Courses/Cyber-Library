---
title: "Netdoc scheme for local file read on the JVM"
description: "netdoc: is a legacy Java URL handler that reads local files, so it serves as an alternative to file:// when that scheme is filtered on a JVM stack."
keywords:
  - netdoc scheme
  - netdoc://
  - Java file read
  - file scheme alternative
  - local file disclosure
---

# Netdoc

`netdoc:` is an old Java URL handler that resolves to a local file read. It predates and parallels `file:`, and because defenders often blocklist `file://` while forgetting `netdoc:`, it is a useful alternative for reading local files on a JVM stack.

## Reading a file

The handler takes a path much like `file:`:

```
netdoc:/etc/passwd
netdoc:///etc/passwd
netdoc:/proc/self/environ
```

On a Java client that still exposes the handler, this returns the file contents through the same channel that reflected the fetch, reaching the same targets as the [File](file.md) scheme: configuration with credentials, environment, source, and keys.

## Why it exists as a separate page

The only reason to use `netdoc:` over `file:` is evasion. A scheme allowlist or blocklist written for `http`/`https`/`file` commonly omits the legacy handler, so when `file://` is rejected but the stack is Java, `netdoc:` reaches the filesystem anyway. It belongs in the rotation alongside [File](file.md) and [JAR](jar.md): when one Java-reachable local scheme is filtered, try the others before concluding local read is unavailable.

## Tools

- **curl**: manual `netdoc:` probing to read a local path on a JVM stack.
- Manual testing with Burp Repeater and crafted payloads.

## References

- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
