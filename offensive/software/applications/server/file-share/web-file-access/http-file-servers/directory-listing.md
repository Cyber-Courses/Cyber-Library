---
title: "Directory listing: enumerating files on an HTTP file server"
description: "Abusing an enabled directory listing (autoindex) on an HTTP file server to browse and enumerate the files it serves, discovering backups, source, configs, and other sensitive files that were not meant to be indexed."
keywords:
  - directory listing
  - autoindex
  - index of
  - file disclosure
  - enumeration
---

# Directory listing

When a web server or file-server application has directory listing enabled and no index file is present, it renders a browsable list of the directory's contents. That discloses every file in the path, including backups, archives, source, and configuration files that were never meant to be linked or indexed.

```bash
# Spot autoindex pages and spider them
curl -s http://<target>/files/ | grep -i 'Index of'
feroxbuster -u http://<target>/ -x bak,zip,old,txt,conf
```

## Exploitation notes

- The classic "Index of /" page exposes files by browsing; combine with content discovery to find listed directories.
- Look specifically for backups (`.bak`, `.zip`, `.old`), source, and config files whose contents hold credentials.
- A listed upload directory is a hint that upload is possible; see [File upload to RCE](file-upload-to-rce.md).

## References

- [HackTricks: pentesting web](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
- [Apache mod_autoindex](https://httpd.apache.org/docs/current/mod/mod_autoindex.html)
