---
title: "Discovery and methods: finding WebDAV and its enabled verbs"
order: 4
description: "WebDAV is detected from the DAV response header and the verbs an OPTIONS request reports. PROPFIND then lists collections and files, and testing PUT reveals whether writes are allowed and which uploaded extensions the server will execute. This enumeration defines exactly which WebDAV attack applies to a target."
keywords:
  - options
  - propfind
  - dav header
  - davtest
  - allowed methods
---

# Discovery and methods

WebDAV enumeration establishes whether DAV is present, which verbs are enabled, and what writes achieve. The `DAV` header in an OPTIONS response confirms WebDAV and its compliance class; the `Allow` header lists the verbs. PROPFIND enumerates collections and files (WebDAV's listing), and a controlled PUT test shows whether uploads are accepted and, crucially, which uploaded extensions the server executes versus serves inert. That last point decides whether an upload is code execution or just a file.

```bash
# verbs and DAV support
curl -s -X OPTIONS http://<target>/ -i | grep -iE 'allow:|dav:'
# PROPFIND listing (Depth: 1 lists the immediate collection)
curl -s -X PROPFIND http://<target>/ -H 'Depth: 1' --data '' | grep -oE '<D:href>[^<]+'
# automated: which extensions can be uploaded and which execute
davtest -url http://<target>/
cadaver http://<target>/                       # interactive DAV client (ls, put, move)
```

Read the `davtest` output specifically for which extensions both uploaded successfully and executed; that is the list of usable payload types for [PUT upload to RCE](put-upload-to-rce.md).

## Exploitation notes

- The `DAV` header presence and a verb list including `PUT`/`MKCOL`/`MOVE` mark a writable WebDAV worth attacking; a read-only DAV (only PROPFIND/GET) still enables listing and traversal.
- `davtest` is the fastest way to learn the executable-extension set; where PUT of `.php`/`.jsp`/`.aspx` is blocked but another executable type (or MOVE-rename) works, that is the path.
- PROPFIND listing reveals files and collections that are not otherwise linked, like a directory listing, feeding both looting and the traversal/upload targets.
- Enumerate as each identity you have (anonymous and any credentials), since verb availability often differs by authentication, which motivates the [authentication bypass](authentication-bypass.md) step.

## Tools

- [davtest](https://github.com/cldrn/davtest)
- [cadaver](http://www.webdav.org/cadaver/)

## References

- [RFC 4918: OPTIONS and PROPFIND](https://datatracker.ietf.org/doc/html/rfc4918)
- [HackTricks: WebDAV](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/put-method-webdav)
