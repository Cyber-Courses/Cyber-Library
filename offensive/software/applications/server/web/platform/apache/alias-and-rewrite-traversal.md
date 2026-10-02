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

A directory alias with permissive traversal handling, or an `AliasMatch` whose regex does not anchor or sanitize the captured path, lets `..` sequences reach the parent:

```apache
Alias /downloads /var/data
AliasMatch "^/files/(.*)$" "/srv/files/$1"
```

The captured group (`$1`) is unconstrained, so traversal in the URL maps above the intended root:

```
GET /downloads../httpd.conf HTTP/1.1
GET /files/../../etc/passwd HTTP/1.1
GET /files/..%2f..%2fetc%2fpasswd HTTP/1.1
```

As with the nginx off-by-slash, an alias prefix without a trailing slash concatenates the following characters (including `..`) straight onto the target path.

## mod_rewrite

`RewriteRule` that composes a path from captured segments is the same class:

```apache
RewriteRule ^/static/(.*)$ /srv/static/$1 [L]
```

With `$1` unconstrained, `/static/../../etc/passwd` resolves above `/srv/static`. Rules using `%{REQUEST_URI}` or passthrough to the filesystem without a `..` guard, and proxying rewrites (`[P]`) that build an upstream path, extend the surface (the proxy case overlaps [reverse proxy and edge](../reverse-proxy-and-edge/index.md)).

## Exploitation

- Start from a known mapped prefix (from asset URLs or a leaked `.htaccess`/config), then append `..`, `../`, and encoded variants (`..%2f`, `%2e%2e%2f`) directly after the prefix.
- Target the parent first (source and config usually sit one level up: `../config.php`, `../.env`), then climb toward `/etc`.
- Combine with [double-decode](../iis/double-decode-and-unicode-traversal.md)-style encoding when a single `..` is filtered.
- Send with `curl --path-as-is` so the client does not pre-normalize the payload.

## Tools

- **curl --path-as-is** / Burp Intruder with traversal wordlists.

## References

- Apache httpd: mod_alias, mod_rewrite
- PortSwigger Web Security Academy: Path traversal
