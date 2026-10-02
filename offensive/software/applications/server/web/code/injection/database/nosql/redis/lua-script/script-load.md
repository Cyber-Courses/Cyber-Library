---
title: "SCRIPT LOAD: caching a script to obtain its SHA1"
description: "SCRIPT LOAD compiles and caches a Lua script without running it and returns its SHA1, seeding the cache for a later EVALSHA in a load-then-exec workflow."
keywords:
  - SCRIPT LOAD
  - EVALSHA
  - SHA1 digest
  - script cache seeding
  - load then exec
---

# SCRIPT LOAD

`SCRIPT LOAD <script>` compiles a Lua script, stores it in the server's script cache, and returns its SHA1 digest, all **without executing it**. It is the staging half of the scripting workflow: the script sits ready in the cache, keyed by `SHA1(script_body)`, waiting to be fired by [EVALSHA](evalsha.md).

## The load-then-exec workflow

Offensively, `SCRIPT LOAD` is valuable precisely because it separates staging from execution. An attacker who reaches a sink that permits `SCRIPT LOAD` can place a malicious, parameterized script into the cache and record the returned digest:

```
SCRIPT LOAD "return redis.call(unpack(ARGV))"
# -> "5f2e...c8"
```

The loaded body is a generic command dispatcher: `unpack(ARGV)` spreads however many arguments are supplied into `redis.call`, so it invokes whatever command name and arguments are passed at execution time with the correct arity (a fixed `ARGV[1],ARGV[2],ARGV[3]` form would pass a trailing `nil` and Redis rejects nil arguments). A single cached script then drives many **script-allowed** commands through `EVALSHA`:

```
EVALSHA 5f2e...c8 0 KEYS *
EVALSHA 5f2e...c8 0 GET sessions:admin
EVALSHA 5f2e...c8 0 SET sessions:admin forged-value
```

The dispatcher is still bound by the scripting sandbox: `noscript` commands such as `CONFIG`, `SLAVEOF`/`REPLICAOF`, and `DEBUG` are rejected inside the script, so an admin-config or write-to-disk path (see [CONFIG SET abuse](../config-set-abuse.md)) must be driven by direct commands, not through this loaded dispatcher.

Because the digest is deterministic, the attacker can compute it offline from the intended body and begin issuing `EVALSHA` immediately after the load succeeds, without reading the load's response. This is useful over blind or one-directional channels such as an SSRF-smuggled RESP connection (see [Command injection](../command.md)), where command output may not return but the staged script still executes.

## Seeding for a split-sink chain

The load and the execution need not share a connection or an injection point. Where one request can trigger `SCRIPT LOAD` and a different request can trigger `EVALSHA`, the cache is shared server-wide, so a script staged through the first is runnable through the second. This decouples the two halves of the attack and lets a short-lived write primitive seed a payload that a later, more limited primitive invokes.

## Inspecting and persistence

`SCRIPT EXISTS <sha1> [sha1 ...]` reports whether given digests are cached (returning `1` or `0` for each), useful to confirm a stage landed before firing it. The cache is in-memory and cleared by `SCRIPT FLUSH` or a server restart, so a staged digest may need re-loading after either. Loading the same body again is idempotent: it returns the same SHA1 and leaves the cache state unchanged, so re-seeding is cheap.

## References

- [Redis: SCRIPT LOAD](https://redis.io/docs/latest/commands/script-load/)
- [Redis: SCRIPT EXISTS](https://redis.io/docs/latest/commands/script-exists/)
- [Redis: EVALSHA](https://redis.io/docs/latest/commands/evalsha/)
