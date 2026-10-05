---
title: "HTTP file servers: attacking directory-serving web apps"
description: "HTTP file servers expose a directory tree for download and sometimes upload, from a language's built-in server to products like HFS and various NAS web UIs. The surface is directory listing that reveals the tree, path traversal to read files outside the served root, and file upload that becomes code execution when the upload path is served or interpreted."
keywords:
  - http file server
  - directory listing
  - path traversal
  - file upload
  - hfs
---

# HTTP file servers

An HTTP file server exposes a directory tree over HTTP for browsing and download, and often for upload. They range from a one-line built-in server (Python's `http.server`, PHP's built-in server) to purpose-built applications (HFS HTTP File Server, various NAS and appliance web UIs). The surface is consistent: directory listing that maps the served tree and reveals files, path traversal that escapes the served root to read arbitrary files, and upload functionality that becomes code execution when the uploaded file lands in a path the server serves or interprets.

```bash
# fingerprint the server and look for listing/upload
curl -sI http://<target>:<port>/              # Server header identifies the product
curl -s http://<target>:<port>/               # directory listing present?
curl -s -X OPTIONS http://<target>:<port>/ -i # methods (PUT => upload)
```

## Subtopics

- **[Directory listing](directory-listing.md)**: mapping the served tree.
- **[Path traversal](path-traversal.md)**: reading files outside the served root.
- **[File upload to RCE](file-upload-to-rce.md)**: turning upload into code execution.
- **[Known server exploits](known-server-exploits.md)**: product-specific file-server vulnerabilities.

## References

- [OWASP: path traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [OWASP: unrestricted file upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
