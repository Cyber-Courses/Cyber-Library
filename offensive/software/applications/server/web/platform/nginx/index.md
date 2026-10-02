---
title: "nginx misconfiguration: alias, FastCGI, proxy_pass, and slash handling"
description: "nginx-specific platform misconfigurations: the location/alias off-by-slash traversal, FastCGI/PHP-FPM wiring that executes uploads, variable proxy_pass SSRF, and merge_slashes/normalization gaps."
keywords:
  - nginx misconfiguration
  - alias off by slash
  - fastcgi_split_path_info
  - proxy_pass ssrf
  - merge_slashes
---

# nginx

nginx is a static server, a FastCGI gateway, and a reverse proxy at once, and its misconfigurations cluster around each role. Fingerprint it from `Server: nginx`, its default error pages, and `location`/`proxy` behavior, then work this checklist. nginx config is copy-pasted constantly, so the same handful of mistakes recur across targets.

## What to check

- **`alias` off-by-slash**: a `location` without a trailing slash plus an `alias` with one concatenates following characters (including `..`) onto the target path, leaking the parent directory.
- **FastCGI/PHP-FPM wiring**: `fastcgi_split_path_info` plus `cgi.fix_pathinfo` executing files that merely contain `.php` in the path, and exposed FPM sockets.
- **Variable `proxy_pass`**: an upstream built from request data (`$host`, an argument) turns the proxy into an SSRF vehicle.
- **Slash and normalization**: `merge_slashes off`, encoded-slash handling, and `location` prefix matching that disagrees with the filesystem or an upstream.

## Pages

- **[Alias off-by-slash](alias-off-by-slash.md)**: the `location`/`alias` traversal into the parent directory.
- **[FastCGI and PHP-FPM](fastcgi-and-php-fpm.md)**: `fastcgi_split_path_info` upload execution, FPM exposure, and `SCRIPT_FILENAME` control.
- **[Variable proxy_pass SSRF](variable-proxy-pass-ssrf.md)**: user-influenced upstream selection and internal routing.
- **[merge_slashes and normalization](merge-slashes-and-normalization.md)**: slash and prefix handling that bypasses `location`-based rules.

## References

- nginx documentation: location, alias, fastcgi_split_path_info, proxy_pass, merge_slashes
- Orange Tsai: nginx off-by-slash and parser research
