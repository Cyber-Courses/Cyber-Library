---
title: "File scheme for local file read via SSRF"
description: "The file:// handler makes a URL client read a local path instead of a network resource, returning the contents of files the service account can open."
keywords:
  - file scheme
  - file://
  - local file read
  - /etc/passwd
  - SSRF file disclosure
---

# File

When a URL client honors `file://`, pointing the sink at a local path makes it read a file from the server's own disk and return the contents through whatever channel reflected the fetch. It is the simplest scheme escalation: a feature meant to fetch a remote URL becomes a local file reader.

## Reading a file

The path follows the scheme, with an empty or `localhost` authority:

```
file:///etc/passwd
file://localhost/etc/passwd
file:///proc/self/environ
file:///proc/self/cwd/app.py
file:///home/app/.ssh/id_rsa
```

On Windows targets the path uses a drive letter:

```
file:///C:/Windows/win.ini
file:///C:/inetpub/wwwroot/web.config
```

Any file the service account can open is reachable: configuration with embedded credentials, environment via `/proc/self/environ`, source code, and private keys are the usual objectives.

## When the response is reflected

`file://` is most useful where the fetched content comes back, for example a preview, an import that echoes what it read, or an error that includes the body. Where the content is not reflected, a blind read still confirms the scheme is honored (through timing or an error that differs for an existing versus a missing path), which is worth knowing because it implies other local schemes such as [Netdoc](netdoc.md) may also work.

## Scheme and parser interaction

Reaching `file://` often requires getting past a scheme allowlist. A client that validates the submitted scheme but follows a [redirect](../query/bypassing-using-a-redirect.md) whose `Location` is `file:///etc/passwd`, or a URL parser that misreads the scheme, lands the read even when `file://` is nominally blocked. Java clients expose `file:` alongside [JAR](jar.md) and [Netdoc](netdoc.md), so if one is filtered the others are worth trying.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
