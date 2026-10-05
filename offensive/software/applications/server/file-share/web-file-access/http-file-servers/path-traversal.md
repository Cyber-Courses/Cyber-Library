---
title: "Path traversal: reading outside an HTTP file server root"
description: "Reading files outside the served root of an HTTP file server with path-traversal sequences in the file parameter or URL, where the server fails to normalize and confine paths, disclosing system files and configuration beyond the intended directory."
keywords:
  - path traversal
  - directory traversal
  - LFI
  - file disclosure
  - ../
---

# Path traversal

HTTP file servers that build a filesystem path from user input and fail to normalize it allow traversal out of the served root. Sequences like `../`, their encoded forms, and absolute paths reach files elsewhere on the host, turning the file server into arbitrary file read. This overlaps the web path-traversal class, applied to the file-serving feature.

```bash
curl --path-as-is 'http://<target>/download?file=../../../../etc/passwd'
curl --path-as-is 'http://<target>/..%2f..%2f..%2fetc%2fpasswd'   # encoded
```

## Exploitation notes

- Try raw, URL-encoded, double-encoded, and overlong UTF-8 traversal sequences, since filters often catch only the literal `../`.
- On Windows file servers, use `..\` and target known config and credential files.
- The deeper treatment of traversal and its bypasses lives in the Web area; this is the file-server-specific application.

## References

- [PortSwigger: path traversal](https://portswigger.net/web-security/file-path-traversal)
- [PayloadsAllTheThings: directory traversal](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal)
