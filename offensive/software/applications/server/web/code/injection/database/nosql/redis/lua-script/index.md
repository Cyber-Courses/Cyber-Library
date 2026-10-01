---
title: "Lua script injection"
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
