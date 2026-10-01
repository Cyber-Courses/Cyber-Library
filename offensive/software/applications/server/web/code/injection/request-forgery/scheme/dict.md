---
title: "DICT scheme for SSRF service interaction"
description: "The dict:// handler opens a TCP connection and sends a single command line, which probes banners and drives line-based internal services such as Redis and memcached."
keywords:
  - dict scheme
  - dict://
  - Redis SSRF
  - memcached
  - service banner
  - line-based protocol
---

# DICT

The `dict://` scheme, supported by curl-backed clients, opens a TCP connection to the host and port in the URL and sends a command line built from the path. It was designed to query dictionary servers, but because it writes an attacker-chosen line to an arbitrary port and returns the reply, it doubles as a way to talk to any line-based internal service.

## Shape of the request

The canonical form carries a command and arguments after the host:

```
dict://127.0.0.1:6379/info
dict://127.0.0.1:11211/stats
```

The client connects to the port and sends the word as a line, then reads the response. curl prepends its own `CLIENT` announcement line before the command, so a line-based service sees one extra line first; Redis and memcached tolerate it and still process the command that follows. Redis replies to `info` with its server block, and memcached replies to `stats`, both of which confirm the service and leak version and configuration.

## One command per request

`dict://` sends a single line per request, so it is ideal for probing and one-shot commands rather than multi-step sequences:

```
dict://127.0.0.1:6379/CONFIG GET dir
dict://127.0.0.1:25/HELO test
```

Each request is a fresh connection and a single command. Reading a service banner this way fingerprints what is listening on a port discovered through [Port](../authority/port.md) scanning. When a sequence of commands is needed (for example the full Redis write-to-disk chain), the single-line limit is the reason to move to [Gopher](gopher.md), which writes a whole crafted stream in one request.

## Probing closed versus open

Because the reply (or the connection error) comes back, `dict://` distinguishes an open port that speaks a known protocol from one that does not, refining a port scan into service identification. Point it at each candidate port and read whether a recognizable banner returns.

## References

- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [curl: DICT protocol](https://curl.se/docs/url-syntax.html)
