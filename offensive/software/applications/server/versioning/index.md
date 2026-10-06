---
title: "Versioning: attacking self-hosted version control"
description: "Attacking self-hosted version control: recovering web-exposed .git and .svn metadata directories to reconstruct source and secrets, reaching the unauthenticated git and Subversion server protocols, and getting code execution through Git hooks and clone-time configuration."
keywords:
  - version control
  - Git
  - Subversion
  - source code disclosure
  - secrets in history
---

# Versioning

A version-control checkout carries far more than the current files: a complete history of every change, the configuration that drives tooling, and, very often, credentials that were committed once and "removed" later but still sit in the object store. Offensive value comes in three directions: reconstructing source and secrets from a metadata directory a web server exposed by accident, reaching a repository server that allows anonymous access, and turning a clone or a push into code execution through the hooks and config the VCS runs automatically.

This area covers the version-control tools themselves. The SaaS code-hosting forges (GitHub, GitLab, Bitbucket) are attacked as vendor-hosted services under [Online > Forges](../../online/forges/index.md).

## Triage

```bash
# A web root that leaked its VCS metadata directory
curl -s -o /dev/null -w '%{http_code}\n' https://<target>/.git/HEAD      # 200 => exposed .git
curl -s -o /dev/null -w '%{http_code}\n' https://<target>/.svn/wc.db     # 200 => exposed .svn (>=1.7)
# A reachable repository server
nmap -p9418,3690 -sV <target>                                            # 9418 git://  3690 svn://
```

A readable `.git`/`.svn` over HTTP routes to the per-tool recovery pages; an open `git://` or `svn://` service routes to the server-protocol pages; a repository you can clone or push to routes to the hooks and config pages.

## Subtopics

- **[Git](git/index.md)**: recovering an exposed `.git`, mining secrets from history, the unauthenticated `git://` daemon, and code execution through hooks and clone-time config.
- **[SVN](svn/index.md)**: recovering an exposed `.svn` working copy, and reaching Subversion history and credentials.

## References

- [Git internals (Pro Git)](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain)
- [Subversion documentation](https://svnbook.red-bean.com/)
- [HackTricks: git and source code disclosure](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/git.html)
