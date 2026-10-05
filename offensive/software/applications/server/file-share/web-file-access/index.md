---
title: "Web file access: attacking file services over HTTP"
description: "Files are increasingly served and received over HTTP: simple HTTP file servers and directory listings, WebDAV for read-write access, and managed file transfer products for business-to-business exchange. Each exposes its own surface, directory listing and traversal, upload-to-execution, authentication bypass, and the heavily-targeted MFT appliance vulnerabilities."
keywords:
  - http file server
  - webdav
  - managed file transfer
  - upload
  - path traversal
---

# Web file access

File sharing has moved onto HTTP, and three forms dominate. Simple HTTP file servers (from a language's built-in server to purpose-built apps) expose directories for download and sometimes upload. WebDAV extends HTTP with read-write verbs, turning a web server into a mountable, writable filesystem. And managed file transfer (MFT) products provide governed business-to-business file exchange through web portals and APIs. Each has a distinct surface: listing and traversal on plain file servers, upload-to-execution and verb abuse on WebDAV, and on MFT the authentication bypasses and injection flaws that have driven some of the largest mass-exploitation campaigns.

```bash
# fingerprint what is serving files over HTTP
curl -sI http://<target>/                      # Server header, allowed methods
curl -s -X OPTIONS http://<target>/ -i | grep -i allow   # WebDAV verbs?
```

## Subtopics

- **[HTTP file servers](http-file-servers/index.md)**: directory listing, traversal, and upload-to-RCE.
- **[WebDAV](webdav/index.md)**: the read-write HTTP extension.
- **[Managed file transfer](managed-file-transfer/index.md)**: the MFT appliances and their exploits.

## References

- [RFC 4918 (WebDAV)](https://datatracker.ietf.org/doc/html/rfc4918)
- [OWASP: unrestricted file upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
