---
title: "Keyword splitting with empty backticks: breaking blocked commands"
description: "Splitting a filtered command keyword with an empty backtick substitution, wh``oami, so the shell removes the empty command and reassembles the keyword, defeating a literal blocklist."
keywords:
  - command injection
  - keyword splitting
  - backticks
  - filter bypass
  - command substitution
  - WAF bypass
---

# Empty backticks

A blocklist that matches a literal keyword such as `whoami` or `cat` can be defeated by inserting an **empty command substitution** inside the word. An empty pair of backticks (`` `` ``) runs "nothing," expands to the empty string, and is deleted during word expansion, leaving the surrounding characters to rejoin into the original keyword. The filter sees `` wh``oami ``; the shell runs `whoami`.

## Why the shell reassembles the word

Backticks are legacy command substitution: `` `cmd` `` runs `cmd` and splices its stdout into the command line. An empty pair runs an empty command, which produces no output and no error, and substitutes to nothing. Crucially, the substitution happens *within* a single word, so the bytes on either side are concatenated after the empty result is removed. `` wh``oami `` tokenizes to the one word `whoami`, which the shell then looks up as a command. A literal-string filter inspecting the raw input never sees the contiguous keyword.

## Payloads

```
wh``oami
ca``t /etc/passwd
i``d
```

The same idea splits any blocked substring, placed anywhere in the word:

```
who``ami
``whoami
whoami``
```

## In an HTTP request

Backticks usually pass through unencoded, but encode them if the application mangles them (`` ` `` is `%60`):

```
wh%60%60oami
```

## Combining with other evasions

Backtick splitting hides the keyword only. If spaces are also filtered, add a space substitute; if separators are filtered, pair with a newline:

```
ca``t${IFS}/etc/passwd
127.0.0.1%0awh``oami
```

## Related forms

The same "insert something that expands to nothing" principle also works with empty `$()` substitution (`who$()ami`), empty quotes (`who''ami`), and backslashes (`who\ami`). Backticks are simply the oldest substitution syntax and are honored by every POSIX shell, including minimal `/bin/sh`.

## Context notes

- This is a **shell** technique; the string must reach `sh -c`, `system()`, or backtick execution.
- Inside double quotes, backticks still trigger substitution, so `` "wh``oami" `` also reassembles, useful when your injection lands in a quoted context.

## Tools

- **[Burp Suite](https://portswigger.net/burp)**: Repeater for crafting empty-backtick payloads.
- **[commix](https://github.com/commixproject/commix)**: tamper scripts automate keyword-splitting bypasses.

## References

- [PayloadsAllTheThings: Command Injection, bypass techniques](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
