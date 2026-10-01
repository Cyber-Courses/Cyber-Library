---
title: "Keyword splitting with double quotes: w\"h\"o\"a\"m\"i"
description: "Breaking a blocked command keyword with empty double-quote pairs, w\"h\"o\"a\"m\"i, so the shell strips the quotes and reassembles the word, bypassing a literal blocklist."
keywords:
  - command injection
  - keyword splitting
  - double quotes
  - filter bypass
  - quote removal
  - WAF bypass
---

# Double quotes

Like single quotes, an **empty pair of double quotes** (`""`) delimits nothing and is removed during quote removal, concatenating its neighbors into one word. Inserting `""` through a filtered keyword, `w"h"o"a"m"i`, reassembles to `whoami` at execution while hiding the contiguous keyword from a literal blocklist.

> **Scope.** For authorized penetration tests, red-team engagements, and CTF labs against systems you own or are contracted to assess. Unauthorized use is unlawful.

## Why the shell removes the quotes

Quote removal runs as a defined expansion step after filtering. `w""h""o""a""m""i` is a single word; each `""` is an empty segment, and the shell deletes every quote character and joins the pieces into `whoami`. A blocklist matching the literal keyword never sees it. Keep the quotes **balanced**, an odd number leaves an unterminated string and the shell reports a syntax error instead of running the command.

## Payloads

```
w""h""o""a""m""i
who""ami
c"a"t /etc/passwd
i"d"
```

## Difference from single quotes

Double quotes are "weak": the shell still performs variable expansion, command substitution, and backslash handling inside them. For *splitting*, that makes no difference, empty pairs vanish the same way. But it also means double quotes can do more than single quotes in an injection:

```
"$(id)"          # substitution still fires inside double quotes
"`whoami`"       # backticks still fire inside double quotes
```

So double-quote splitting is interchangeable with single-quote splitting for hiding a keyword, and additionally composes with substitution when you need execution rather than mere concealment.

## In an HTTP request

URL-encode the quote as `%22` if the application does not accept it raw:

```
w%22h%22oami
```

## Combining with other evasions

Add a space substitute or separator where those are filtered:

```
c"a"t${IFS}/etc/passwd
127.0.0.1;w"h"o"a"m"i
```

## Context and caveats

- This is a **shell** technique (`sh -c`, `system()`, backticks).
- Quotes must be **balanced**; an unmatched `"` leaves an open string and breaks the command.
- If your injection already sits inside a double-quoted context, remember that `$`, `` ` ``, and `\` remain active there—useful for breaking out, but also means a stray `$` or backtick in your payload may be interpreted.

## References

- [PayloadsAllTheThings: Command Injection, bypass techniques](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
