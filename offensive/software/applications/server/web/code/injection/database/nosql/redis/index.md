---
title: "Redis injection"
description: "Command injection via the RESP protocol and abuse of dangerous commands in a Redis key-value store, from CRLF smuggling to CONFIG SET RCE and server-side Lua."
keywords:
  - Redis injection
  - RESP protocol
  - CRLF injection
  - CONFIG SET
  - Lua scripting
---

# Redis

Redis is an in-memory key-value store spoken over **RESP**, a simple CRLF-delimited wire protocol. Injection against Redis is less about query syntax and more about reaching the connection: when attacker-controlled input is embedded in a command, or when SSRF lets an attacker speak to a Redis port directly, raw `\r\n` sequences smuggle additional commands into the stream.

Once commands can be smuggled, Redis offers high-impact primitives: `FLUSHALL` and `KEYS` for mass access, `CONFIG SET` to relocate the dump file and write arbitrary content to disk, `SLAVEOF`/`REPLICAOF` to pull a dataset from a rogue master, `MODULE LOAD` where modules are permitted, and the `EVAL` family for server-side Lua. This subtree covers command smuggling over RESP, the `CONFIG SET` write-to-disk RCE chain, and Lua scripting via `EVAL`, `EVALSHA`, and `SCRIPT LOAD`.

## Tools

- **redis-cli**: the native Redis client for issuing and testing commands.
- **Gopherus**: generate gopher:// RESP payloads for SSRF-smuggled commands.
- **Burp Suite**: intercept and tamper with Redis-backed web requests.

## References

- [Redis: command reference](https://redis.io/docs/latest/commands/)
- [PayloadsAllTheThings: Redis SSRF and command injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
