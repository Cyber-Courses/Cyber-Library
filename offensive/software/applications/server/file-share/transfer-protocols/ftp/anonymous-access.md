---
title: "Anonymous access: reading files from an open FTP server"
description: "Reaching files on an FTP server configured for anonymous access, logging in with the anonymous account and no real password to list directories and download files, a common default on public and misconfigured servers."
keywords:
  - anonymous FTP
  - anonymous login
  - FTP download
  - directory listing
  - unauthenticated
---

# Anonymous access

Many FTP servers allow the `anonymous` (or `ftp`) account with any or no password, a default meant for public downloads but frequently left on sensitive servers. Anonymous login lists directories and downloads files, and where anonymous upload is also enabled, it becomes a drop point.

```bash
ftp <target>                                 # user: anonymous, pass: anything
# or non-interactively
curl ftp://<target>/ --user anonymous:anon   # list
wget -r ftp://anonymous:anon@<target>/       # mirror everything
nmap -p 21 --script ftp-anon <target>        # detect anonymous access
```

## Exploitation notes

- `ftp-anon` flags anonymous access and whether upload is allowed; anonymous read is loot, anonymous write is a foothold.
- Anonymous-writable directories that are also web-served turn into web-shell upload; check whether the FTP root maps to a web root.
- Recursively mirror the server; config, backup, and source files are common on anonymous FTP.

## References

- [Nmap ftp-anon](https://nmap.org/nsedoc/scripts/ftp-anon.html)
- [HackTricks: pentesting FTP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ftp/index.html)
