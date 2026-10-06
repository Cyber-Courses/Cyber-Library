---
title: "Directory listing: mapping the served file tree"
order: 1
description: "When an HTTP file server has no index file or has autoindex enabled, it renders a directory listing that exposes every file and subdirectory. An attacker uses that listing, or forces it, to map the served tree, discover files not meant to be linked (backups, configs, source), and find the upload or traversal points to attack next."
keywords:
  - directory listing
  - autoindex
  - index of
  - file discovery
  - enumeration
---

# Directory listing

A directory listing is the automatic index a web server renders for a directory that has no index file (or where autoindex is explicitly on). It exposes every file and subdirectory by name, which is a reconnaissance gift: files never linked from the application, backups, `.bak` and `.old` copies, configuration, source archives, and logs, all become visible and downloadable. Mapping the listed tree finds the sensitive files to pull and the upload or dynamic endpoints to attack.

```bash
# detect and walk listings
curl -s http://<target>/            # "Index of /" or a file-server UI => listing
# recursively mirror a listed tree
wget -r -np -R 'index.html*' http://<target>/
# force/confirm listing on specific directories (dropping the index file name)
for d in backup old uploads config .git includes; do
  curl -s -o /dev/null -w "%{http_code} $d/\n" http://<target>/$d/; done
```

## Exploitation notes

- Prioritise by name: `backup`, `old`, `.bak`/`.old` suffixes, `config`, `.git`, `uploads`, and archive files (`.zip`, `.tar.gz`, `.sql`) are the high-value finds a listing reveals.
- A listing that exposes `.git/` enables full source recovery (dump the repo), and exposed `.sql`/backup files give data and credentials offline.
- Even where the app links no listing, individual directories may still autoindex; probe likely directory names directly.
- The listing is the map for the other attacks: it locates the writable upload directory ([File upload to RCE](file-upload-to-rce.md)) and shows the structure to aim traversal at ([Path traversal](path-traversal.md)).

## References

- [OWASP: directory indexing](https://owasp.org/www-project-web-security-testing-guide/)
- [Apache/nginx autoindex behaviour](https://httpd.apache.org/docs/current/mod/mod_autoindex.html)
