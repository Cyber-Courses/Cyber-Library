---
title: "Command injection filter bypass by character reconstruction"
order: 5
description: "Rebuilding forbidden characters like / from environment-variable substrings (${HOME:0:1}), tr translation, and printf, so a filter blocking specific characters never sees them in the request."
keywords:
  - command injection
  - character reconstruction
  - filter bypass
  - environment variable substring
  - tr
  - printf
---

# Character reconstruction

Some filters block **specific characters** rather than whole keywords, most often `/`, but also `.`, `;`, or spaces. The counter is to reconstruct the forbidden character from material the shell already holds: a substring of an environment variable, a translation with `tr`, or a byte emitted by `printf`. The shell produces the character at runtime, so it never appears in the request the filter inspects.

## Why the shell normalizes it away

The filter operates on the literal request bytes. If `/` is on the blocklist, `cat /etc/passwd` is rejected. But `${HOME:0:1}` is a parameter expansion whose *value* is `/`, and the filter sees only the letters `H`, `O`, `M`, `E`, braces, digits, and a colon, none of which is a slash. The shell evaluates the expansion before execution and substitutes the real `/`. The blocked character is manufactured from a variable's contents rather than typed, so the literal blocklist has nothing to match.

## Rebuilding `/` from variable substrings

Any variable whose value starts with (or contains) a slash yields one by slicing:

```
${HOME:0:1}          # "/"  (HOME=/home/...)
${PATH:0:1}          # "/"  (PATH=/usr/bin:...)
${PWD:0:1}           # "/"
```

Assemble a full path with no literal slash in the request:

```
cat ${HOME:0:1}etc${HOME:0:1}passwd
cat$IFS${PATH:0:1}etc${PATH:0:1}passwd
```

Pick a different offset/length to carve out other characters a variable happens to contain (`:` from `$PATH`, `.` from a version string, letters for a keyword).

## Rebuilding characters with `tr`

`tr` translates one character set into another, so a benign input character can be turned into a forbidden one at runtime:

```
echo . | tr '.' '/'                  # emits "/"
cat $(echo _etc_passwd | tr '_' '/') # "/etc/passwd" from underscores
```

Here the request contains underscores, not slashes; `tr` performs the substitution inside command substitution, and `cat` receives the reconstructed path.

## Rebuilding characters with printf / octal / hex

`printf` can emit arbitrary bytes from escape sequences, producing a character the filter blocks:

```
cat $(printf '\57')etc$(printf '\57')passwd   # \57 is octal for "/"
printf '\x2f'                                  # hex for "/"
```

Bash ANSI-C quoting does the same inline, with no external binary:

```
cat $'\x2f'etc$'\x2f'passwd                    # $'\x2f' → "/"
```

The escape sequences are plain ASCII in the request; the shell decodes them to the forbidden byte only at execution time.

## Combining reconstructed characters

These primitives compose with no-space and expansion tricks, so a command can avoid several blocked characters at once:

```
cat${IFS}${HOME:0:1}etc${HOME:0:1}passwd
{cat,$'\x2f'etc$'\x2f'passwd}
```

Each slash, each space, and each keyword fragment is produced by the shell, leaving a filter that blocks literal `/` and spaces nothing to catch.

## Operational notes

- Substring expansion `${VAR:0:1}` and `$'\xNN'` ANSI-C quoting are **Bash** features; `tr`/`printf` work in any POSIX shell (subject to those binaries not themselves being filtered).
- Choose a source variable guaranteed to be present, `HOME`, `PATH`, and `PWD` reliably start with `/`.
- The method reconstructs **characters**; combine it with brace/glob/variable techniques when whole command names are also filtered.

## Tools

- **[Burp Suite](https://portswigger.net/burp)**: Repeater for crafting reconstructed-character payloads.
- **[commix](https://github.com/commixproject/commix)**: tamper scripts automate character and space bypasses.

## References

- [PayloadsAllTheThings: Command Injection, Bypass without specific characters](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [Bash Reference Manual: Shell Parameter Expansion](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html)
