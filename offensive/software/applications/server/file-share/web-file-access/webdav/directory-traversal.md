---
title: "Directory traversal: escaping the WebDAV root"
description: "Reading or writing files outside the WebDAV root through path-traversal in the request path or destination header, where the server fails to confine DAV operations, exposing files beyond the intended directory or writing to sensitive paths."
keywords:
  - WebDAV traversal
  - Destination header
  - path traversal
  - file disclosure
  - MOVE COPY
---

# Directory traversal

WebDAV operations take paths in the request URL and in headers like `Destination` (for MOVE and COPY). Where the server does not normalize and confine these, traversal sequences reach files outside the DAV root, enabling read of system files and, through MOVE or PUT, writes to paths the DAV directory should not reach.

```bash
# Read outside the root
curl --path-as-is -X GET 'http://<target>/dav/../../../../etc/passwd'
# Write outside via a traversal Destination on MOVE
curl -X MOVE http://<target>/dav/x.txt -H 'Destination: http://<target>/dav/../../inetpub/wwwroot/x.aspx'
```

## Exploitation notes

- The `Destination` header on MOVE and COPY is a second traversal surface distinct from the request path; test both.
- Combining traversal with PUT or MOVE can place an executable file into a web root outside the DAV directory, reaching execution.
- Use encoded and platform-specific separators as with any traversal; the Web area covers the full bypass set.

## References

- [HackTricks: pentesting WebDAV](https://book.hacktricks.wiki/en/network-services-pentesting/put-method-webdav.html)
- [RFC 4918: WebDAV](https://www.rfc-editor.org/rfc/rfc4918)
