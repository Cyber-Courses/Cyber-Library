---
title: "Platform: attacking the web server and hosting layer, by product"
description: "Offensive techniques against the web platform organized the way an audit works: general exposures that apply to any server, then per-product misconfiguration checklists for Apache, nginx, IIS, and Tomcat/Java, plus reverse-proxy and edge behavior."
keywords:
  - web server exploitation
  - nginx misconfiguration
  - apache misconfiguration
  - iis misconfiguration
  - tomcat
  - reverse proxy
---

# Platform

The **platform** is the web server and the way the service is hosted and exposed: Apache, nginx, IIS, or a Java app server; reverse proxies and CDNs; virtual hosts; the rules that map a URL to a file or a backend; and the handlers that decide whether a file is served, disclosed, or executed. This layer owns a class of vulnerabilities distinct from the application logic ([Code](../code/index.md)) and the language engine ([Runtime](../runtime/index.md)). The test for whether a bug belongs here: *would it still exist if the application source and the language runtime were benign, because the weakness is in server configuration, deployment topology, or how the server resolves and routes requests?*

## How this area is organized

The library's organizing rule is to split by the axis you identify *first* at a given layer, then refine. In application Code that axis is the dangerous primitive; in Runtime it is primitive then language or engine. At the platform layer the thing you identify first is the **product**: you fingerprint the server, then run that product's misconfiguration checklist. So Platform is organized by product (Apache, nginx, IIS, Tomcat), which is the same principle applied one layer out, not a departure from it. A single **general** section holds the product-agnostic exposures (a served `.git`, directory listing, backups, leaked config) so they are not repeated under each product, and a **reverse proxy and edge** section covers behavior that belongs to the proxy role rather than any one product.

The underlying primitives still run through every product, so each page is tagged with its primitive and the table below gives a primitive-first way in, on top of the product-first tree.

## By primitive (cross-reference)

| Primitive | Where it appears |
|-----------|------------------|
| Path traversal / mapping | [Apache alias/rewrite](apache/alias-and-rewrite-traversal.md), [nginx alias off-by-slash](nginx/alias-off-by-slash.md), [nginx slash normalization](nginx/merge-slashes-and-normalization.md), [IIS double-decode](iis/double-decode-and-unicode-traversal.md), [proxy normalization mismatch](reverse-proxy-and-edge/normalization-mismatch.md) |
| Source disclosure | [Apache handler](apache/handler-and-type-mapping.md), [Apache MultiViews](apache/multiviews-and-negotiation.md), [IIS NTFS tricks](iis/ntfs-filename-tricks.md), [general VCS](general/version-control-directories.md), [general backups](general/backup-and-temporary-files.md) |
| Handler / upload to execution | [Apache handler](apache/handler-and-type-mapping.md), [nginx FastCGI/PHP-FPM](nginx/fastcgi-and-php-fpm.md), [IIS handlers](iis/handlers-and-web-config.md) |
| SSRF / routing | [nginx variable proxy_pass](nginx/variable-proxy-pass-ssrf.md), [proxy_pass and misrouting](reverse-proxy-and-edge/index.md), [origin exposure](reverse-proxy-and-edge/origin-exposure.md) |
| Deployment RCE | [Tomcat Manager](tomcat-and-java/manager-and-host-manager.md), [AJP/Ghostcat](tomcat-and-java/ajp-ghostcat.md) |
| Information exposure | [general](general/index.md) (status endpoints, VCS, backups, listing, config) |
| Header / identity trust | [edge header trust](reverse-proxy-and-edge/edge-header-trust.md) |

## Fingerprint first

Before the checklists, identify the server: `Server`/`X-Powered-By` headers, default error pages, cookie names (`JSESSIONID` = Java, `ASP.NET_SessionId` = IIS), header ordering and casing, favicon hashes, and behavior on malformed requests. The fingerprint selects the section.

## Sections

- **[General](general/index.md)**: exposures on any server, status endpoints, `.git`/VCS, backups, directory listing, leaked config.
- **[Apache](apache/index.md)**: handler/type mapping, `Alias`/`mod_rewrite` traversal, MultiViews negotiation.
- **[nginx](nginx/index.md)**: `alias` off-by-slash, FastCGI/PHP-FPM wiring, variable `proxy_pass` SSRF, slash normalization.
- **[IIS](iis/index.md)**: NTFS filename tricks, double-decode/Unicode traversal, handlers and `web.config`.
- **[Tomcat and Java](tomcat-and-java/index.md)**: AJP/Ghostcat, Manager deployment, path-parameter traversal.
- **[Reverse proxy and edge](reverse-proxy-and-edge/index.md)**: edge/origin normalization mismatch, origin exposure, edge header trust.

## References

- Apache httpd and nginx documentation (configuration references)
- PortSwigger Web Security Academy: Information disclosure, Access control, SSRF
