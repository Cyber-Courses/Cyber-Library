---
title: "Exposed version control directories: reconstructing source from .git, .svn, and .hg"
order: 2
description: "Finding and dumping web-served version-control directories to recover full application source, history, and secrets, then reconstructing the working tree offline."
keywords:
  - git exposure
  - .git directory
  - git-dumper
  - svn exposure
  - source disclosure
---

# Version control directories

Deploying by `git clone`/`git pull` into the web root, or copying a working directory, leaves the `.git` (or `.svn`, `.hg`) folder under the document root. When the server serves those files as static content, an attacker downloads the repository metadata and reconstructs the **entire source tree and its history**, including secrets committed and later "removed".

## Detecting exposure

Probe for the metadata files that must exist in a repo:

```
GET /.git/HEAD            HTTP/1.1     # "ref: refs/heads/main"
GET /.git/config          HTTP/1.1     # remotes, maybe creds in URLs
GET /.svn/wc.db           HTTP/1.1
GET /.hg/requires         HTTP/1.1
```

A 200 with `ref: refs/heads/...` on `/.git/HEAD` confirms a served Git directory. Directory listing being off does not protect you: the file names inside `.git` are predictable, so the tree can be walked without it.

## Dumping and reconstructing

Even without directory listing, Git's object model lets you walk from `HEAD` to every object. Automated tools do this:

```bash
git-dumper https://target/.git/ ./loot
# or
python3 gitdumper.py https://target/.git/ ./loot && cd loot && git checkout -- .
```

`git-dumper` fetches `HEAD`, refs, packed objects, and loose objects, then checks out the working tree. From there:

```bash
git log --all --oneline          # full history, including deleted content
git show <commit>:config.php      # recover a file from any revision
git log -p | grep -iE 'password|secret|api[_-]?key|token'
```

Secrets that were committed then deleted remain in history, so grep every revision, not just the tip.

## Exploitation

- Recovered source reveals the exact injection/deserialization sinks, framework secret keys (signed-cookie and ViewState forgery), and database and third-party credentials.
- `.git/config` and the packed-refs sometimes contain remote URLs with embedded tokens; the logs (`.git/logs/HEAD`) reveal committer emails and branch activity.
- For `.svn`, parse `wc.db` (SQLite) for the pristine file list and pull the cached pristine copies; for `.hg`, fetch the store and reconstruct.

## Tools

- **git-dumper**, **gitdumper.py** (GitTools), **svn-extractor**.

## References

- OWASP WSTG: Review webserver metafiles / source code disclosure
- GitTools project
