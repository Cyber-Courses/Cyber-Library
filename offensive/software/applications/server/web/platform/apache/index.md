---
title: "Apache httpd misconfiguration: handlers, aliases, and content negotiation"
description: "Apache-specific platform misconfigurations: over-broad handler and type mappings, .htaccess override abuse, Alias and mod_rewrite traversal, and MultiViews content negotiation disclosure."
keywords:
  - apache misconfiguration
  - AddHandler
  - htaccess
  - mod_rewrite
  - MultiViews
---

# Apache

Apache httpd's flexibility, per-directory `.htaccess` overrides, extension-based handlers, and a rich module set, is also its misconfiguration surface. Fingerprint it from the `Server: Apache` header, default error pages, and `.htaccess`/`mod_*` behavior, then work this checklist.

## What to check

- **Handler and type mapping**: does a handler match an inner extension (`shell.php.jpg`), is a directory's handler set to execute uploads, or is the PHP handler missing so scripts return as source?
- **.htaccess overrides**: where `AllowOverride` is permissive and a directory is writable (uploads), an attacker-supplied `.htaccess` changes handlers, rewrites, and auth.
- **Alias and rewrite**: do `Alias`/`AliasMatch`/`mod_rewrite` rules build filesystem paths from unconstrained URL captures, allowing traversal into the parent of the mapped directory?
- **MultiViews**: is content negotiation enabled, letting an attacker request `page` and have Apache pick `page.php`/`page.bak`, enumerating and disclosing files?

## Pages

- **[Handler and type mapping](handler-and-type-mapping.md)**: `AddHandler`/`AddType`/`SetHandler` execution and source-disclosure mistakes, `.htaccess` abuse, and double-extension upload execution.
- **[Alias and rewrite traversal](alias-and-rewrite-traversal.md)**: `Alias`/`AliasMatch`/`mod_rewrite` path mapping that escapes the intended directory.
- **[MultiViews and content negotiation](multiviews-and-negotiation.md)**: abusing `mod_negotiation` to enumerate and disclose files.

## References

- Apache httpd documentation: mod_mime, mod_alias, mod_rewrite, mod_negotiation, AllowOverride
- OWASP WSTG: Testing for application platform configuration
