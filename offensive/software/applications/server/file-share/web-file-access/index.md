---
title: "Web file access: attacking HTTP-based file sharing"
description: "Attacking web-based file access: HTTP file servers and their directory listing, traversal, and upload flaws, WebDAV's methods and PUT upload, and managed file transfer appliances (MOVEit, GoAnywhere, Serv-U, CrushFTP) whose auth-bypass and injection flaws have driven mass-data-theft campaigns."
keywords:
  - web file sharing
  - HTTP file server
  - WebDAV
  - managed file transfer
  - upload
---

# Web file access

Files are increasingly shared over HTTP: lightweight file-server software, WebDAV extensions to the web server, and managed file transfer (MFT) appliances for business exchange. These are attacked like web applications, through directory listing and traversal, file upload to code execution, and authentication bypass, with MFT appliances standing out as a repeated source of mass-breach exploit chains.

## Subtopics

- **[HTTP file servers](http-file-servers/index.md)**: file-server software and autoindex.
- **[WebDAV](webdav/index.md)**: the HTTP file-management extension.
- **[Managed file transfer](managed-file-transfer/index.md)**: enterprise MFT appliances.

## References

- [HackTricks: pentesting web](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
- [PayloadsAllTheThings: file upload](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)
