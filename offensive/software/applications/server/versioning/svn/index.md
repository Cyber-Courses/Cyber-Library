---
title: "SVN: attacking Subversion version control"
description: "Attacking Subversion: recovering a web-exposed .svn working copy to reconstruct source and metadata from the wc.db database and pristine store, and reaching Subversion repository history and credentials over svn and HTTP."
keywords:
  - SVN
  - Subversion
  - .svn exposure
  - svnserve
  - source code disclosure
---

# SVN

Subversion keeps a working copy's metadata in a `.svn` directory. Since version 1.7 that is a single SQLite database (`.svn/wc.db`) plus a `pristine` store of the unmodified file content, while older layouts scattered a `.svn` folder with `entries` and `text-base` copies into every directory. Either way, a `.svn` directory that a web server serves gives up the full source and its metadata, and the repository server behind it often allows anonymous history access that recovers deleted files and committed credentials.

## Triage

```bash
curl -s -o /dev/null -w '%{http_code}\n' https://<target>/.svn/wc.db          # 200 => exposed (>=1.7)
curl -s -o /dev/null -w '%{http_code}\n' https://<target>/.svn/entries        # 200 => exposed (legacy)
nmap -p3690 -sV <target>; svn ls svn://<target>/ 2>/dev/null                  # svnserve, anonymous read
```

## Pages

- **[Exposed .svn directory](exposed-svn-directory.md)**: rebuild source and metadata from a web-served working copy.
- **[Repository history and credentials](repository-history-and-credentials.md)**: reach Subversion history to recover removed files and credentials.

## References

- [Version Control with Subversion (the SVN book)](https://svnbook.red-bean.com/)
- [HackTricks: pentesting SVN / source disclosure](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
