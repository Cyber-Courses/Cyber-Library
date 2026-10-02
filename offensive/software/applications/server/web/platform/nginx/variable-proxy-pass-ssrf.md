---
title: "nginx variable proxy_pass SSRF: user-influenced upstream selection"
description: "Exploiting nginx proxy_pass built from request data: variable upstreams that disable validation and let an attacker redirect the proxied request to internal services and cloud metadata."
keywords:
  - proxy_pass ssrf
  - variable proxy_pass
  - nginx ssrf
  - cloud metadata
  - gopher
---

# Variable proxy_pass SSRF

When nginx builds its `proxy_pass` upstream from request data, an attacker redirects the proxied request to an internal service, turning the trusted edge into an SSRF vehicle. Because the request originates from nginx (inside the network), it reaches hosts and ports the attacker cannot touch directly.

## The dangerous pattern

Using a variable in `proxy_pass` disables nginx's normal upstream resolution-at-startup and can incorporate attacker input:

```nginx
location /proxy/ {
    proxy_pass http://$host$request_uri;      # attacker-controlled Host -> arbitrary upstream
}
location /fetch/ {
    set $target $arg_url;
    proxy_pass http://$target;                # ?url=169.254.169.254/... -> cloud metadata
}
```

Setting `Host:` or the `url` argument to an internal address makes nginx connect there:

```
GET /fetch/?url=169.254.169.254/latest/meta-data/iam/security-credentials/ HTTP/1.1
GET /proxy/ HTTP/1.1
Host: 127.0.0.1:6379
```

Classic targets: cloud metadata (`169.254.169.254`), localhost-only admin panels, and internal APIs (Elasticsearch, Consul, Docker, Kubernetes, actuator).

## Path and join issues

Even without a variable upstream, a trailing-slash mismatch between `location` and `proxy_pass`, or characters forwarded unescaped, can alter the upstream path, reaching unintended backend routes. Encoded CRLF or spaces that nginx passes through have, on some versions, injected into the upstream request line.

## Pivoting to raw protocols

Where an SSRF-reachable component speaks more than HTTP, escalate with `gopher://` to send arbitrary bytes to an internal TCP service: a FastCGI packet to reach [PHP-FPM](fastcgi-and-php-fpm.md) on `:9000`, or a Redis/SMTP command sequence, crafted with **Gopherus**.

## Exploitation

- Find proxy endpoints that take a URL/host/path (`/proxy`, `/fetch`, `/render`, image or webhook fetchers) and point them at `127.0.0.1`, link-local metadata, and internal hostnames.
- Read cloud metadata for credentials, then pivot off-host.
- Confirm blind SSRF with an out-of-band interaction (Burp Collaborator), then target specific internal services.

## Tools

- **Gopherus**, Burp Collaborator.

## References

- nginx: proxy_pass (variable form caveats)
- PortSwigger Web Security Academy: SSRF
