---
title: "Spaceless command injection: executing shell commands without the space character"
order: 1
description: "Producing whoami/id when the space character is filtered, using ${IFS}, $IFS$9, brace expansion, input redirection, and tab substitutes to separate a command from its arguments."
keywords:
  - command injection
  - filter bypass
  - no space
  - IFS
  - brace expansion
  - WAF bypass
---

# Spaceless payloads

Many command-injection filters block the ASCII space (`0x20`), assuming that without it an attacker cannot separate a command from its arguments. The shell, however, offers several other ways to produce a field separator, and it applies them *after* the filter has inspected the raw bytes. A blocklist that greps for a literal space, or for `cat /etc/passwd` as one token, never sees the separator the shell eventually synthesizes.

## Why the shell fills the gap

Word splitting in POSIX shells is driven by the `IFS` variable, which defaults to space, tab, and newline. Any construct that expands to one of those characters, or that separates words structurally, removes the need for a typed space. The filter sees `${IFS}`; the shell sees whitespace.

## `${IFS}` and variants

The canonical substitute expands `IFS` directly between tokens:

```
cat${IFS}/etc/passwd
whoami${IFS}
```

`$IFS` alone often fails because the shell greedily reads the following characters as part of the variable name. Append a positional parameter that is almost always empty to terminate it cleanly:

```
cat$IFS$9/etc/passwd
```

`$9` (an unset positional argument) expands to nothing, so `$IFS$9` collapses to a single separator. `${IFS%??}` and `$IFS$1` behave similarly.

## Brace expansion

A comma-separated brace list is expanded into separate words with no space in the source:

```
{cat,/etc/passwd}
{whoami,}
{id,}
```

The shell splits the braces into `cat` `/etc/passwd` as distinct `argv` entries before execution.

## Input redirection

The `<` operator feeds a file to a command's stdin and needs no space around the filename:

```
cat</etc/passwd
sort</etc/passwd
```

For commands that read stdin, this delivers file contents while avoiding both the space and an explicit path argument.

## Tabs and URL-encoded whitespace

A literal tab is also in the default `IFS`. In an HTTP request it is sent URL-encoded, which a space-only filter does not match:

```
cat%09/etc/passwd
```

`%09` (tab) and, in some contexts, `%0b`/`%0c` reach the shell as whitespace. Combine with keyword-splitting tricks where the keyword itself is filtered.

## Tools

- **[Burp Suite](https://portswigger.net/burp)**: Repeater for crafting space-free payloads.
- **[commix](https://github.com/commixproject/commix)**: tamper scripts automate no-space bypasses.

## References

- [PayloadsAllTheThings: Command Injection, bypass without space](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [GTFOBins](https://gtfobins.github.io/)
