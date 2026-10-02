---
title: "EVALSHA: running cached scripts by SHA1"
description: "EVALSHA executes a Lua script already cached in Redis by its SHA1 digest, letting an attacker re-invoke a script they loaded or one already present in the cache."
keywords:
  - EVALSHA
  - SCRIPT LOAD
  - SHA1 digest
  - Redis script cache
  - cached script reuse
---

# EVALSHA

`EVALSHA <sha1> <numkeys> [key ...] [arg ...]` runs a Lua script that is already cached in the server's script cache, identified by the SHA1 digest of its body. It is the bandwidth-saving twin of `EVAL`: the client sends a 40-character hash instead of the full script, and Redis runs the cached copy with identical semantics.

## Relationship to SCRIPT LOAD and EVAL

A script enters the cache two ways. `EVAL` caches its script as a side effect of running it, and `SCRIPT LOAD <script>` caches a script **without** running it and returns its SHA1 (see [SCRIPT LOAD](script-load.md)). Either way, the digest is deterministic: it is `SHA1(script_body)`, so the same script always has the same key. After a load, the script can be fired repeatedly by digest:

```
SCRIPT LOAD "return redis.call('KEYS','*')"
# -> "a9b1c3...e7"
EVALSHA a9b1c3...e7 0
```

Because the digest depends only on the body, an attacker can precompute it offline. If a script with a known body is already cached, `EVALSHA` runs it without the attacker ever supplying the text.

## Reuse of attacker-loaded scripts

The common offensive flow is load-then-exec across two reachable sinks. Where one injection point allows `SCRIPT LOAD` (or an earlier `EVAL`) and another allows `EVALSHA`, the attacker stages a malicious script once and triggers it later:

```
SCRIPT LOAD "return redis.call('SET', KEYS[1], ARGV[1])"
# cache now holds a key-writing script under its SHA1
EVALSHA <sha1> 1 sessions:admin "forged-value"
```

The staged script is parameterized through `KEYS`/`ARGV`, so a single cached body drives multiple actions by varying the arguments on each `EVALSHA`.

Note the ceiling: `EVAL`/`EVALSHA` can only invoke **script-allowed** commands. `CONFIG` (SET/GET), `SLAVEOF`/`REPLICAOF`, and `DEBUG` carry the `noscript` flag and are rejected inside a Lua script with "This Redis command is not allowed from script", so the write-to-disk RCE sequence in [CONFIG SET abuse](../config-set-abuse.md) cannot be staged through Lua and must be issued as **direct** commands. What `EVALSHA` stages is data-plane manipulation (`GET`/`SET`/`KEYS`/`MGET`/`SCAN`) over a blind or split sink.

## NOSCRIPT and re-seeding

If the digest is not cached, Redis replies with a `NOSCRIPT` error. That signals the script must be seeded first, with `SCRIPT LOAD` or a full `EVAL`, before `EVALSHA` will run it. A cache flush (`SCRIPT FLUSH`) or a server restart clears all cached scripts, so a staged `EVALSHA` chain may need re-seeding after either event. When only `EVALSHA` is reachable and the body is unknown, enumerating useful digests is impractical; the practical path is to obtain a `SCRIPT LOAD` or `EVAL` sink to control the body first.

## Tools

- **redis-cli**: SCRIPT LOAD a body and fire it by digest with EVALSHA.
- **Burp Repeater**: trigger the cached-script execution through the sink.

## References

- [Redis: EVALSHA](https://redis.io/docs/latest/commands/evalsha/)
- [Redis: SCRIPT LOAD](https://redis.io/docs/latest/commands/script-load/)
- [Redis: scripting with Lua](https://redis.io/docs/latest/develop/interact/programmability/eval-intro/)
