---
title: "Status and info endpoints: server-status, server-info, and stub_status exposure"
order: 1
description: "Harvesting operational dashboards left public: Apache mod_status and mod_info, nginx stub_status, and PHP-FPM status pages, which leak live requests, internal paths, and configuration."
keywords:
  - server-status
  - mod_status
  - server-info
  - stub_status
  - php-fpm status
---

# Status and info endpoints

Web servers and app runtimes ship operational dashboards that are meant for localhost or an admin network but are frequently reachable from the internet. They leak live traffic, internal paths, module lists, and configuration, all without authentication.

## Apache mod_status and mod_info

`mod_status` publishes a live server dashboard; when `/server-status` is not restricted it shows the requests currently being served, including other users' URLs (with query strings), client IPs, and vhosts:

```
GET /server-status HTTP/1.1
GET /server-status?refresh=1 HTTP/1.1
```

`mod_info` at `/server-info` dumps the full parsed configuration: loaded modules, vhost layout, and directory directives, which is a complete map of the platform. Both commonly leak because a `Location` block restricting them was scoped to one vhost but the module is global.

## nginx stub_status

`ngx_http_stub_status_module` exposes connection and request counters:

```
GET /nginx_status HTTP/1.1
GET /status HTTP/1.1
GET /basic_status HTTP/1.1
```

Less rich than Apache's, but it confirms nginx, reveals load, and the presence of the endpoint signals a copy-pasted config worth probing further.

## PHP-FPM and app-server status

PHP-FPM exposes a pool status page when `pm.status_path` is set and routed:

```
GET /status HTTP/1.1
GET /fpm-status HTTP/1.1
GET /fpm-ping HTTP/1.1
```

It leaks the FPM pool name, request counts, slow requests, and sometimes the script paths being executed. Equivalent dashboards exist for other stacks (uWSGI stats, Passenger status); probe the common paths and any referenced in leaked config.

## Exploitation

- Request the common paths directly, and retry with a spoofed `X-Forwarded-For: 127.0.0.1` or `Host: localhost` where access is IP- or host-gated at the server.
- On `server-status`, poll with `?refresh=1` and scrape the request table over time to capture session tokens and IDs that other users pass in query strings.
- Feed recovered internal paths into the [nginx FastCGI/PHP-FPM](../nginx/fastcgi-and-php-fpm.md) and [Apache handler](../apache/handler-and-type-mapping.md) techniques.

## Tools

- **curl** / Burp; **nuclei** has templates for these exposures.

## References

- Apache httpd: mod_status, mod_info
- nginx: ngx_http_stub_status_module; PHP-FPM: pm.status_path
