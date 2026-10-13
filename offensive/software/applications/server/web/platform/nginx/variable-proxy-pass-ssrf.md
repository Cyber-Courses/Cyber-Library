---
title: "nginx variable proxy_pass SSRF: user-influenced upstream selection"
order: 3
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
    proxy_pass http://$host$request_uri;      # Host chooses the upstream HOST (port dropped)
}
location /raw/ {
    proxy_pass http://$http_host$request_uri; # $http_host keeps the port from the Host header
}
location /fetch/ {
    set $target $arg_url;
    proxy_pass http://$target;                # ?url=169.254.169.254/... -> full host:port control
}
```

A detail that trips people up: `$host` is the normalized server name **without** the port, so `Host: 127.0.0.1:6379` with `$host` connects to `127.0.0.1:80`, not Redis on 6379. To control the port from the Host header you need `$http_host` (the raw header value), and to control host and port freely the `$arg_url` form is simplest:

```
GET /fetch/?url=127.0.0.1:6379/ HTTP/1.1
GET /fetch/?url=169.254.169.254/latest/meta-data/iam/security-credentials/ HTTP/1.1
GET /raw/ HTTP/1.1
Host: 127.0.0.1:9000
```

Classic targets: cloud metadata (`169.254.169.254`), localhost-only admin panels, and internal HTTP APIs (Elasticsearch, Consul, Docker, Kubernetes, actuator).

## Path and join issues

Even without a variable upstream, a trailing-slash mismatch between `location` and `proxy_pass`, or characters forwarded unescaped, can alter the upstream path, reaching unintended backend routes. Encoded CRLF or spaces that nginx passes through have, on some versions, injected into the upstream request line.

## Reach is HTTP(S) only

nginx's HTTP `proxy_pass` speaks only HTTP and HTTPS, so this primitive cannot emit raw bytes to a non-HTTP service: you cannot turn a variable `proxy_pass` into a `gopher://` FastCGI/Redis/SMTP payload. It reaches internal **HTTP** services and cloud metadata, which is already high-impact. Raw-byte pivots (a FastCGI packet to [PHP-FPM](fastcgi-and-php-fpm.md), a Redis command stream) require a *different* SSRF sink whose URL client supports `gopher://`, such as an application-level `curl`/library fetch, not nginx itself. Many internal services (Redis, Memcached) are still reachable here if they tolerate an HTTP request line, but the clean raw-protocol route needs the gopher-capable sink.

## Exploitation

- Find proxy endpoints that take a URL/host/path (`/proxy`, `/fetch`, `/render`, image or webhook fetchers) and point them at `127.0.0.1`, link-local metadata, and internal hostnames.
- Read cloud metadata for credentials, then pivot off-host.
- Confirm blind SSRF with an out-of-band interaction (Burp Collaborator), then target specific internal services.

## Tools

- Burp Repeater/Collaborator (confirm blind SSRF out-of-band). Gopherus applies only to a separate gopher-capable SSRF sink, not to nginx `proxy_pass`.

## References

- nginx: proxy_pass (variable form caveats)
- PortSwigger Web Security Academy: SSRF
