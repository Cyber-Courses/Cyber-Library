---
title: "File upload to RCE: turning an upload into code execution"
order: 2
description: "An HTTP file server that accepts uploads becomes code execution when an uploaded file is placed where the server interprets it: a script in a web-executable directory, a file whose extension or content-type is mishandled, or an upload combined with traversal to control the destination path. The planted file then runs server-side when requested."
keywords:
  - file upload
  - web shell
  - extension bypass
  - content-type
  - unrestricted upload
---

# File upload to RCE

Upload functionality is dangerous whenever the uploaded file can end up somewhere the server will execute it. The direct case is uploading a server-side script (`.php`, `.jsp`, `.aspx`) into a directory the web server interprets, giving a web shell. Where the application tries to restrict uploads, bypasses abound: forbidden extensions evaded with alternates or casing, content-type or magic-byte checks fooled, double extensions, null bytes, and trailing characters. And an upload combined with path traversal lets the attacker choose the destination, placing an executable file into a web-served path even if uploads are meant to be confined.

```bash
# plain upload of a web shell where the dir is executable
curl -s -F 'file=@shell.php' http://<target>/upload
curl -s 'http://<target>/uploads/shell.php?cmd=id'          # trigger it
# extension / content-type bypasses when filtering exists
#   shell.php.jpg | shell.pHp | shell.phtml/.php5 | trailing dot/space (Windows)
curl -s -F 'file=@shell.phtml;type=image/jpeg' http://<target>/upload
# upload + traversal to control the write path (escape the uploads dir)
curl -s -F 'file=@shell.php;filename=../../shell.php' http://<target>/upload
```

## Exploitation notes

- Two conditions must meet: the file must be stored and the storage location must be interpreted. Confirm where uploads land (the listing or the response often reveals the path) and whether that path executes scripts.
- Filter bypasses are extension-based (`.phtml`, `.php5`, case, double extension), content-based (prepend valid magic bytes, set an image content-type), or environment-specific (Windows trailing dot/space, `::$DATA`); try them systematically when a naive upload is rejected.
- If the uploads directory is not executable, use traversal in the filename to place the file in one that is, or chain with a server config overwrite (`.htaccess` to make a directory execute a chosen extension).
- Non-script payloads matter too: overwriting a config, cron, or `authorized_keys` via upload+traversal reaches execution without a web shell.

## References

- [OWASP: unrestricted file upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [PortSwigger: file upload vulnerabilities](https://portswigger.net/web-security/file-upload)
