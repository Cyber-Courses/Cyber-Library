---
title: "Keyword splitting with positional parameters: who$@ami and $0"
description: "Breaking a blocked keyword by inserting positional-parameter expansions that evaluate to nothing, who$@ami, who${@}ami, and abusing $0 as a shell invocation, so a literal blocklist misses the keyword."
keywords:
  - command injection
  - keyword splitting
  - positional parameters
  - "$@"
  - "$0"
  - filter bypass
---

# Positional parameters ($@, $0)

The shell's special parameters expand to nothing in a non-interactive context with no arguments, which makes them useful as an invisible separator inside a word. `$@` (all positional parameters) and `$*` expand to the empty string when none are set, so `who$@ami` reassembles to `whoami` after expansion. `$0` is separately useful: it holds the name of the shell itself and can be executed directly.

## `$@` and `$*` as an empty splitter

In `sh -c "..."` with no extra arguments, `$@` expands to nothing. Placed inside a keyword it vanishes and the neighbors join:

```
who$@ami
who${@}ami
c$@at /etc/passwd
i$@d
```

The brace form `${@}` is more robust because it bounds the parameter name explicitly, preventing the shell from reading following letters as part of it. A blocklist matching `whoami` sees only `who$@ami`.

## Why it expands to empty

`$@` is the list of positional arguments `$1 $2 ...`. A web sink that calls `sh -c "<string>"` passes no positional parameters, so the list is empty and the expansion contributes zero characters to the word. The same holds for `$*`, `$1`, `$9`, and other unset positionals.

## `$0` as a shell handle

`$0` expands to the name the shell was invoked as (typically `sh` or `bash`). That makes it a filtered-keyword-free way to spawn a shell or pipe a command into one:

```
echo id | $0
$0 -c id
```

Here no literal `sh`/`bash` keyword appears in the payload, yet `$0` resolves to the interpreter and runs the staged command.

## In an HTTP request

`$`, `@`, `{`, `}` usually pass unencoded; encode as needed (`$`=`%24`, `@`=`%40`):

```
who%24%40ami
```

## Combining with other evasions

Pair with a space substitute or separator:

```
c$@at${IFS}/etc/passwd
127.0.0.1;who${@}ami
```

## Context and caveats

- This is a **shell** technique (`sh -c`, `system()`, backticks); in a pure `argv` call `$@` is a literal string.
- If the sink passes user-controlled positional arguments to the shell, `$@`/`$1` may not be empty, prefer a positional known to be unset such as `$9`.

## Tools

- **[Burp Suite](https://portswigger.net/burp)**: Repeater for crafting positional-parameter payloads.
- **[commix](https://github.com/commixproject/commix)**: tamper scripts automate keyword-splitting bypasses.

## References

- [PayloadsAllTheThings: Command Injection, bypass techniques](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
