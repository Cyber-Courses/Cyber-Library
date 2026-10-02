---
title: "Directory listing: autoindex and directory browsing as an enumeration source"
description: "Using server directory listing (Apache autoindex, nginx autoindex, IIS directory browsing) to enumerate files, discover unreferenced content, and locate backups, uploads, and config."
keywords:
  - directory listing
  - autoindex
  - directory browsing
  - indexes
  - enumeration
---

# Directory listing

When a directory has no index file and the server is configured to list its contents, requesting the directory returns a browsable index of every file in it. This hands an attacker the exact names of files they would otherwise have to guess: backups, uploads, logs, config, and unreferenced scripts.

## Recognizing it

A directory URL returning an HTML index titled "Index of /..." (Apache/nginx) or a plain file table (IIS) is directory browsing:

```
GET /uploads/ HTTP/1.1
GET /backup/  HTTP/1.1
GET /.git/    HTTP/1.1     # listing also makes VCS dumping trivial
```

Apache's `Options +Indexes`, nginx's `autoindex on;`, and IIS directory browsing each produce this. It is often enabled on a subdirectory (uploads, assets, exports) even when the root has an index file.

## Why it is useful

- **Names you would never guess**: a listing of `/uploads/` reveals every uploaded filename, exposing other users' documents and any planted webshell path.
- **Backups and dumps**: a listing turns the guesswork of [backup files](backup-and-temporary-files.md) into a directory read.
- **Timestamps and sizes**: the index reveals modification times and sizes, useful for spotting recently changed files and large archives/dumps.
- **Recursive mapping**: walk every listed subdirectory to build a complete file map without a wordlist.

## Exploitation

- Enumerate likely listed directories (`/uploads`, `/files`, `/backup`, `/export`, `/static`, `/tmp`, `/logs`, `/.git`) directly.
- Recursively mirror a listed tree:

  ```bash
  wget -r -np -nH --reject "index.html*" https://target/uploads/
  ```

- Sort the index by date/size (Apache supports `?C=M;O=D`) to surface the newest or biggest files first, then pull source, dumps, and secrets.

## Tools

- **wget -r** / **curl**; browser for quick triage.

## References

- Apache httpd: mod_autoindex, Options Indexes
- nginx: ngx_http_autoindex_module
