---
title: "Exposed .git directory: reconstructing source from a web-served repository"
description: "Recovering a web-exposed .git directory into the full source tree: detecting the exposure, downloading loose and packed objects with and without directory listing, rebuilding the working tree with git-dumper, and reading config and refs for credentials and internal detail."
keywords:
  - .git exposure
  - git-dumper
  - source code disclosure
  - packed objects
  - git checkout
---

# Exposed .git directory

When a deployment copies a working tree to the web root without excluding `.git/`, the server hands out the entire object database. Because Git is content-addressed, that is enough to rebuild every tracked file at the latest commit (and usually the whole history), recovering source that was never meant to be public along with the `config`, remote URLs, and any secret that was committed.

## Fingerprint first

`.git/HEAD` is the cheapest tell: it is tiny, always present, and its contents are unmistakable.

```bash
curl -s https://target/.git/HEAD        # => "ref: refs/heads/main"  (a ref line, not an HTML 404 page)
curl -s https://target/.git/config      # remote URLs, often credentials in the URL
```

A `ref:` line (or a 40-char SHA) confirms it. If the response is an HTML error page with a `200`, the server has a catch-all and you must verify by fetching a known object path rather than trusting the status code.

## With directory listing

If the server lists directories (`Options +Indexes`), mirror the whole folder:

```bash
wget -r -np -R 'index.html*' https://target/.git/
cd target && git checkout -- .          # materialize the working tree from the recovered objects
```

## Without directory listing

Usually listing is off, so you fetch the known Git paths and walk the object graph. `git-dumper` automates exactly this: read `.git/HEAD` and the refs, pull `objects/info/packs` and the pack index/pack files, then fetch loose objects by hash as they are discovered.

```bash
git-dumper https://target/.git/ loot/
cd loot && git log --oneline && git checkout -- .
```

Doing it by hand, the paths that matter are `HEAD`, `config`, `packed-refs`, `logs/HEAD` (the reflog, which names commits no ref points to), `objects/info/packs`, each `objects/pack/pack-<sha>.idx` and `.pack`, and loose objects at `objects/<first2>/<rest38>`. A loose object is zlib-compressed; `git cat-file -p <sha>` decodes it once it is in a local `.git`.

```bash
# the reflog reveals commit SHAs even for branches that were deleted or force-pushed
curl -s https://target/.git/logs/HEAD
git cat-file -p <sha>                   # read a recovered commit/tree/blob
```

## What to read first

- `config` and `logs/HEAD` give remote URLs and the full ref history, including deleted branches.
- `git log -p` across the recovered history is where the real loot is, feed it to [secrets in history](secrets-in-history.md).
- The source itself is the map for the next stage: hardcoded endpoints, credentials, and the application's own vulnerabilities.

## Tools

- [git-dumper](https://github.com/arthaud/git-dumper): recover a `.git` without directory listing.
- [GoGitDumper / goop](https://github.com/nyancrimew/goop): alternative dumpers that brute-force common object paths.

## References

- [Git internals: the object database](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)
- [HackTricks: exposed .git](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/git.html)
