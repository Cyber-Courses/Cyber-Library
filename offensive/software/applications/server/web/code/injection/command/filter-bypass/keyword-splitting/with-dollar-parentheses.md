---
title: "Keyword splitting with empty command substitution: who$()ami"
description: "Breaking a blocked keyword by inserting an empty $() command substitution, who$()ami, so the shell runs nothing, removes it, and reassembles the keyword, bypassing a literal blocklist."
keywords:
  - command injection
  - keyword splitting
  - command substitution
  - "$()"
  - filter bypass
  - WAF bypass
---

# Empty command substitution (`$()`)

Modern command substitution uses `$( ... )`. An **empty** substitution, `$()`, runs no command, produces no output, and is removed during expansion, so like empty quotes or backticks it can be slipped inside a filtered keyword to break the signature. `who$()ami` runs an empty command between `who` and `ami`, then collapses to `whoami`.

> **Scope.** For authorized penetration tests, red-team engagements, and CTF labs against systems you own or are contracted to assess. Unauthorized use is unlawful.

## Why the shell reassembles the word

`$(cmd)` executes `cmd` and substitutes its stdout. With nothing between the parentheses there is nothing to run; the substitution yields the empty string. Because it occurs inside a single word, the surrounding bytes concatenate once the empty result is spliced in. `who$()ami` tokenizes to the one word `whoami`, which the shell then resolves as a command. A literal blocklist inspecting the raw bytes never sees the contiguous keyword.

## Payloads

```
who$()ami
ca$()t /etc/passwd
i$()d
```

The insertion can go anywhere in the word, and more than once:

```
$()whoami
who$()a$()mi
```

## `$()` vs backticks

`$()` and `` `` `` are equivalent for this purpose; `$()` is preferred because it nests cleanly and is less likely to be mangled by intermediate processing. Both are "insert something that expands to nothing" tricks, in the same family as `''`, `""`, `\`, and `$@`.

## In an HTTP request

`$`, `(`, `)` generally pass unencoded; encode if required (`$`=`%24`, `(`=`%28`, `)`=`%29`):

```
who%24%28%29ami
```

## Combining with other evasions

The substitution hides the keyword only. Add a space substitute or separator where those are filtered:

```
ca$()t${IFS}/etc/passwd
127.0.0.1;who$()ami
```

## Context and caveats

- This is a **shell** technique (`sh -c`, `system()`, backticks); in a pure `argv` call `$()` is a literal string.
- `$()` substitution also fires inside **double** quotes, so `"who$()ami"` reassembles too, handy when your injection lands in a double-quoted context.
- A **non-empty** `$()` is the execution primitive itself (`$(id)`); here the empty form is used purely for concealment.

## References

- [PayloadsAllTheThings: Command Injection, bypass techniques](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
