---
title: "Apache Alias and mod_rewrite traversal: mapping outside the intended directory"
description: "Exploiting Apache Alias, AliasMatch, and mod_rewrite rules that build filesystem paths from unconstrained URL captures, letting a request escape the mapped directory into source, config, or system files."
keywords:
  - apache alias
  - AliasMatch
  - mod_rewrite traversal
  - RewriteRule
  - path traversal
---

# Alias and rewrite traversal

Apache maps URL prefixes to directories with `Alias`/`AliasMatch` and rewrites them with `mod_rewrite`. When a rule builds a filesystem path from a captured URL segment without constraining `..`, a request walks into the parent directory and reaches files outside the intended folder.

## Alias and AliasMatch

Plain `Alias` is **not** vulnerable to the nginx off-by-slash: Apache matches `Alias` on complete path segments and normalizes the URL path (collapsing `../`) before alias processing, so `/downloads../x` does not match `Alias /downloads` and decoded dot-segments are already resolved. The Apache vector is instead an **`AliasMatch` or `RewriteRule` whose regex capture is substituted into a filesystem path without constraint**, combined with encoded separators that survive normalization:

```apache
AliasMatch "^/files/(.*)$" "/srv/files/$1"
```

Here `$1` is unconstrained. Because Apache decodes and normalizes `../` in the path before matching, a literal `/files/../../etc/passwd` is collapsed first; the working payload relies on **encoded** separators that are not normalized when `AllowEncodedSlashes` is `On`/`NoDecode`, so the encoded `..%2f` reaches the capture and then the filesystem:

```
GET /files/..%2f..%2f..%2fetc%2fpasswd HTTP/1.1
```

Without `AllowEncodedSlashes NoDecode` (the default is `Off`, which rejects `%2f`), test whether the server accepts encoded slashes at all first; if it rejects them, this `AliasMatch` path is not exploitable and the `RewriteRule` cases below (or a proxy mismatch) are the remaining avenues.

## mod_rewrite

`RewriteRule` that composes a path from captured segments is the same class, with the same caveat: a rule matching the normalized `REQUEST_URI` never sees a literal `../` (it was collapsed), so the traversal must come from an unnormalized source or encoded separators:

```apache
RewriteRule ^/static/(.*)$ /srv/static/$1 [L]
```

The exploitable variants are rules that match against `%{THE_REQUEST}` (the raw request line, which is not decoded or normalized, so raw and encoded `../` both survive into the capture), rules that run with `AllowEncodedSlashes NoDecode` so `..%2f` reaches `$1`, and proxying rewrites (`[P]`) that build an upstream path (the proxy case overlaps [reverse proxy and edge](../reverse-proxy-and-edge/index.md)).

## Exploitation

- Start from a known mapped prefix (from asset URLs or a leaked `.htaccess`/config), then put traversal inside the captured path, favoring encoded separators (`..%2f`, `%2e%2e%2f`, `%2e%2e/`) since Apache collapses literal `../` before matching.
- First confirm the server accepts encoded slashes at all (`AllowEncodedSlashes`); if `%2f` is rejected, this path is closed and the `%{THE_REQUEST}`/proxy cases remain.
- Target the parent first (source and config usually sit one level up: `../config.php`, `../.env`), then climb toward `/etc`.
- Combine with [double-decode](../iis/double-decode-and-unicode-traversal.md)-style encoding when a single encoded `..` is filtered.
- Send with `curl --path-as-is` so the client does not pre-normalize the payload.

## Tools

- **curl --path-as-is** / Burp Intruder with traversal wordlists.

## References

- Apache httpd: mod_alias, mod_rewrite
- PortSwigger Web Security Academy: Path traversal
