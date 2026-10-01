---
title: "Keyword splitting with single quotes: w'h'o'a'm'i"
description: "Breaking a blocked command keyword with empty single-quote pairs—w'h'o'a'm'i—so the shell strips the quotes and reassembles the word, bypassing a literal blocklist."
keywords:
  - command injection
  - keyword splitting
  - single quotes
  - filter bypass
  - quote removal
  - WAF bypass
---

# Single-quote splitting

Single quotes in the shell delimit a literal string, but an **empty pair of single quotes** (`''`) delimits nothing. During quote removal the shell deletes the quotes and concatenates whatever surrounds them into one word. Sprinkling `''` through a filtered keyword—`w'h'o'a'm'i`—therefore reassembles to `whoami` at execution time while never appearing as the contiguous keyword in the raw input.

> **Scope.** For authorized penetration tests, red-team engagements, and CTF labs against systems you own or are contracted to assess. Unauthorized use is unlawful.

## Why the shell removes the quotes

Quote removal is a defined step of shell expansion that runs after the filter has already inspected the bytes. `w'h'o'a'm'i` tokenizes as a single word; each `''` contributes an empty literal segment, and the shell strips all quote characters, joining the segments into `whoami`. A blocklist matching the literal string `whoami` sees only the quoted form and lets it pass.

## Payloads

Empty pairs between every character, or just enough to break the signature:

```
w'h'o'a'm'i
'w'h'o'a'm'i'
c'a't /etc/passwd
i'd'
```

You can also wrap single characters as non-empty quotes, which is equivalent:

```
'whoami'          # one quoted word, still runs whoami
```

## In an HTTP request

Single quotes commonly pass unencoded; URL-encode as `%27` if needed:

```
w%27h%27oami
```

## Combining with other evasions

Quote splitting hides the keyword only. Add a space substitute or separator where those are also filtered:

```
c'a't${IFS}/etc/passwd
127.0.0.1;w'h'o'a'm'i
```

## Context and caveats

- This is a **shell** technique (`sh -c`, `system()`, backticks).
- Quotes must be **balanced**. An odd number of single quotes leaves an open literal and the shell waits for a closing quote, breaking the command—always pair them.
- If your injection point already sits **inside** single quotes (e.g. `sh -c 'ping '$x''`), you must first close that literal with a `'` before this trick applies; see the quoting-context discussion on the shell-metacharacter page.
- Single quotes are stronger than double quotes: inside `'...'` no expansion occurs, so this form is purely for splitting, not for triggering substitution.

## References

- [PayloadsAllTheThings: Command Injection — bypass techniques](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
