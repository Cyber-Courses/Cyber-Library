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

`svn log` enumerates revisions and `svn cat` retrieves any file at any revision, which recovers content that no longer exists at HEAD:

```bash
svn log -v svn://<target>/repo                      # revisions, with the paths each one changed
svn cat svn://<target>/repo/config/db.ini@42        # the file as it existed at revision 42
svn export svn://<target>/repo ./repo-head          # the full current tree in one step
```

Read the `-v` log for paths marked `D` (deleted): the file is gone from HEAD, so pin the URL to a revision where it still existed with a peg revision (`@42`), not just `-r`. A plain `svn cat -r 42 <url>` resolves the URL's peg at HEAD first, where the path no longer exists, and fails; `<url>@42` (optionally `-r 42` as well) locates the deleted node and brings it back. A "remove the password" commit is recovered exactly this way.

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
