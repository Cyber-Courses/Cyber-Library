---
title: "File upload to RCE: executing through an HTTP file server upload"
description: "Turning an HTTP file server's upload feature into code execution by uploading a server-executable file (PHP, JSP, ASPX) where the upload directory is served and executable, bypassing extension and content-type filters, then requesting the uploaded file."
keywords:
  - file upload
  - web shell
  - extension bypass
  - RCE
  - upload directory
---

# File upload to RCE

Where an HTTP file server accepts uploads and the upload directory is both web-served and allowed to execute scripts, uploading a web shell in the server's language yields code execution when the file is requested. The work is bypassing the upload filters (extension allowlists, content-type checks, magic-byte checks) and locating the uploaded file's URL.

```bash
# Upload a shell, bypassing a naive extension filter
curl -F 'file=@shell.phar' http://<target>/upload       # alternate PHP extension
curl -F 'file=@shell.php;type=image/png' http://<target>/upload   # content-type spoof
curl 'http://<target>/uploads/shell.phar?cmd=id'         # trigger
```

## Exploitation notes

- Match the payload to the server runtime: `.php`/`.phar` for PHP, `.jsp`/`.jspx` for Java, `.aspx` for IIS; the directory must be allowed to execute that type.
- Common bypasses: alternate extensions, double extensions, content-type spoofing, trailing characters, and null or path tricks in the filename.
- The exhaustive upload-bypass technique set lives in the Web area; this applies it to file-server upload.

## References

- [PortSwigger: file upload vulnerabilities](https://portswigger.net/web-security/file-upload)
- [PayloadsAllTheThings: upload insecure files](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)
