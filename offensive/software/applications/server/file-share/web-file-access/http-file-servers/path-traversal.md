---
title: "Path traversal: reading files outside the served root"
description: "An HTTP file server that builds a filesystem path from the request without properly canonicalising it allows path traversal: ../ sequences, encoded variants, or absolute paths in the URL or a filename parameter reach files outside the served root. This yields arbitrary file read, recovering configs, credentials, and source at the server process's privilege."
keywords:
  - path traversal
  - directory traversal
  - ../
  - url encoding
  - arbitrary file read
---

# Path traversal

A file server maps a request to a file on disk, and if it concatenates the served root with the requested path without canonicalising and bounding the result, an attacker escapes the root with traversal sequences. The classic `../` walks up directories; encoded forms (`%2e%2e%2f`, double-encoding, overlong UTF-8), mixed separators on Windows (`..\`), and absolute paths bypass naive filters. The result is arbitrary file read limited only by the server process's privileges, recovering system files, application configuration, credentials, and source.

```bash
# basic and encoded traversal in the path or a download parameter
curl -s --path-as-is http://<target>/../../../../etc/passwd
curl -s 'http://<target>/download?file=../../../../etc/passwd'
curl -s 'http://<target>/download?file=%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd'  # url-encoded
curl -s 'http://<target>/download?file=....//....//etc/passwd'                   # filter bypass
# Windows targets
curl -s --path-as-is 'http://<target>/..\..\..\windows\win.ini'
```

## Exploitation notes

- Use `--path-as-is` so curl does not collapse the `../` before sending; many filters are bypassed by encoding (`%2e%2e%2f`), double-encoding, or the `....//` pattern that survives a single naive strip.
- Target high-value files for the platform: `/etc/passwd` and `/etc/shadow`, application config (`web.config`, `.env`, framework settings with DB creds), SSH keys, and the app's own source for further flaws.
- A traversal in a download/`file=` parameter is as useful as one in the path; test both.
- Read is bounded by the server process privilege; a file server running as root or a service account reaches more. Pair with any upload for write-side impact ([File upload to RCE](file-upload-to-rce.md)).

## References

- [OWASP: path traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [PortSwigger: directory traversal](https://portswigger.net/web-security/file-path-traversal)
