---
title: "HTTP file servers: attacking file-server software and autoindex"
description: "Attacking HTTP file servers: directory listing that discloses files, path traversal that reads outside the served root, file upload that reaches code execution, and named remote code execution flaws in common file-server software such as HTTP File Server (HFS)."
keywords:
  - HTTP file server
  - HFS
  - autoindex
  - directory listing
  - file upload
---

# HTTP file servers

HTTP file servers range from a web server's directory autoindex (Apache, nginx) to dedicated software like HTTP File Server (HFS) and filebrowser. They expose files over HTTP and often allow uploads. Attacks disclose files through listing and traversal, turn upload into code execution, and exploit named flaws in the server software itself.

## Subtopics

- **[Directory listing](directory-listing.md)**: enumerating exposed files.
- **[Path traversal](path-traversal.md)**: reading outside the served root.
- **[File upload to RCE](file-upload-to-rce.md)**: turning upload into execution.
- **[Known server exploits](known-server-exploits.md)**: named RCE in file-server software.

## References

- [HackTricks: pentesting web](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
- [PayloadsAllTheThings: directory traversal](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal)
