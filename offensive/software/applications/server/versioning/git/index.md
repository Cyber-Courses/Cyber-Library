---
title: "Git: attacking the Git version-control tool"
description: "Attacking Git itself: recovering a web-exposed .git directory into full source, mining commit history and dangling objects for secrets, cloning from an unauthenticated git daemon, and getting code execution through hooks and clone-time configuration."
keywords:
  - Git
  - .git exposure
  - git daemon
  - git hooks
  - secrets in history
---

# Git

Git stores a repository as a content-addressed object database under `.git/`: commits, trees, and blobs keyed by SHA-1, plus refs, a packed-object store, and a `config`. Everything an attacker wants follows from that layout. If a web server serves `.git/`, the whole tree and its history can be rebuilt offline. If history holds a credential that was later deleted, it is still in the object store. If a repository server is anonymous, it can be cloned. And because Git runs hooks and honors repository config automatically, a clone or a push can become code execution.

## Triage

```bash
curl -s -o /dev/null -w '%{http_code}\n' https://<target>/.git/HEAD     # 200 + "ref: refs/heads/..." => exposed
nmap -p9418 -sV <target>                                                # git:// daemon
git ls-remote git://<target>/<repo>                                     # anonymous clone test
```

## Pages

- **[Exposed .git directory](exposed-git-directory.md)**: rebuild the full source tree and config from a web-served `.git`.
- **[Secrets in history](secrets-in-history.md)**: recover credentials from commit history, dangling objects, and the reflog.
- **[Daemon](git-daemon.md)**: clone private repositories from an unauthenticated `git://` service.
- **[Hooks and config execution](hooks-and-config-execution.md)**: code execution through hooks and clone-time configuration.

## References

- [Git internals: plumbing and porcelain](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain)
- [git-dumper](https://github.com/arthaud/git-dumper)
