---
title: "Lua script injection"
order: 3
description: "Redis runs server-side Lua through the EVAL family, giving an attacker who controls a script body or its arguments a scripting engine inside the database."
keywords:
  - Redis Lua
  - EVAL
  - EVALSHA
  - SCRIPT LOAD
  - server-side scripting
---

# Lua script

Redis embeds a Lua interpreter and runs scripts server-side through the `EVAL` family of commands. A script can call back into Redis with `redis.call()`, loop over keys, transform values, and return arbitrary results, all atomically on the server. When an attacker controls a script body, or the arguments passed to one, that scripting engine becomes an injection surface inside the database.

Scripts reach the server three ways: `EVAL` runs a script supplied inline, `SCRIPT LOAD` caches a script and returns its SHA1 digest, and `EVALSHA` runs a previously cached script by that digest. The three are linked: a script loaded once with `SCRIPT LOAD` can be re-run cheaply by SHA1, and an attacker who can load a script can later invoke it. This subtree covers each command as an injection vector.

## Pages

- **[EVAL](eval.md)**: EVAL runs a Lua script inside Redis; injecting into the script body or its arguments lets an attacker drive redis.call() and reach every key server-side.
- **[EVALSHA](evalsha.md)**: EVALSHA executes a Lua script already cached in Redis by its SHA1 digest, letting an attacker re-invoke a script they loaded or one already present in the ca...
- **[SCRIPT LOAD](script-load.md)**: SCRIPT LOAD compiles and caches a Lua script without running it and returns its SHA1, seeding the cache for a later EVALSHA in a load-then-exec workflow.

## Tools

- **redis-cli**: issue EVAL, SCRIPT LOAD, and EVALSHA against the server.
- **Gopherus**: smuggle scripting commands over an SSRF RESP connection.
- **Burp Suite**: intercept and tamper with Redis-backed web requests.

## References

- [Redis: scripting with Lua](https://redis.io/docs/latest/develop/interact/programmability/eval-intro/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
