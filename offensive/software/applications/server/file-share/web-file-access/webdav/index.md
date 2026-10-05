---
title: "WebDAV: attacking the HTTP file-management extension"
description: "Attacking WebDAV, the HTTP extension that adds file management to a web server: discovering it and enumerating its methods, bypassing weak authentication, traversing outside the DAV root, and abusing the PUT method to upload a web shell for code execution."
keywords:
  - WebDAV
  - PROPFIND
  - PUT
  - mod_dav
  - IIS WebDAV
---

# WebDAV

WebDAV extends HTTP with methods (PROPFIND, MKCOL, PUT, MOVE, COPY) that let clients manage files on the server, implemented by IIS WebDAV and Apache `mod_dav`. It is attacked by discovering it and its allowed methods, bypassing weak authentication, traversing outside its root, and, most directly, using PUT to upload an executable file.

## Subtopics

- **[Discovery and methods](discovery-and-methods.md)**: detecting DAV and its allowed methods.
- **[Authentication bypass](authentication-bypass.md)**: reaching DAV past weak auth.
- **[Directory traversal](directory-traversal.md)**: escaping the DAV root.
- **[PUT upload to RCE](put-upload-to-rce.md)**: uploading a web shell.

## References

- [HackTricks: pentesting WebDAV](https://book.hacktricks.wiki/en/network-services-pentesting/put-method-webdav.html)
- [RFC 4918: WebDAV](https://www.rfc-editor.org/rfc/rfc4918)
