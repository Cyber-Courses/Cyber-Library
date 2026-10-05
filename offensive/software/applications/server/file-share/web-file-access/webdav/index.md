---
title: "WebDAV: attacking the read-write HTTP extension"
description: "WebDAV extends HTTP with verbs for listing, creating, moving, and writing files, turning a web server into a mountable, writable filesystem. The offensive surface is discovering the enabled verbs, bypassing weak authentication, traversing outside the intended collection, and using PUT or MOVE to place an executable file in a web-interpreted path for code execution."
keywords:
  - webdav
  - propfind
  - put
  - dav verbs
  - move
---

# WebDAV

WebDAV (Web Distributed Authoring and Versioning) adds verbs to HTTP, PROPFIND (list and read properties), PUT (write a file), MKCOL (make a collection), MOVE, COPY, DELETE, LOCK, so a web server becomes a mountable, writable filesystem. That read-write capability is the attack surface. The moves are discovering which verbs are enabled and what they expose, bypassing authentication that is often weak or inconsistently applied, traversing outside the intended collection, and, most impactful, using PUT (sometimes combined with MOVE to dodge an extension filter) to plant a server-executable file for code execution.

```bash
# detect WebDAV and its verbs
curl -s -X OPTIONS http://<target>/ -i | grep -iE 'allow|dav'   # DAV header + verbs
davtest -url http://<target>/                 # tests PUT and which extensions execute
```

## Subtopics

- **[Discovery and methods](discovery-and-methods.md)**: finding WebDAV and its enabled verbs.
- **[Authentication bypass](authentication-bypass.md)**: reaching DAV verbs past weak auth.
- **[Directory traversal](directory-traversal.md)**: escaping the intended collection.
- **[PUT upload to RCE](put-upload-to-rce.md)**: planting an executable file via PUT/MOVE.

## References

- [RFC 4918 (WebDAV)](https://datatracker.ietf.org/doc/html/rfc4918)
- [HackTricks: WebDAV](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/put-method-webdav)
