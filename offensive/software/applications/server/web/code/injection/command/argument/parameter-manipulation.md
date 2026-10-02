---
title: "Argument injection and parameter manipulation without a shell"
description: "Turning a user value into argv so a called binary parses it as a flag, no shell needed, plus worst-fit and fullwidth-character tricks and the escapeshellarg/escapeshellcmd bypass concept."
keywords:
  - argument injection
  - parameter manipulation
  - argv injection
  - worst-fit mapping
  - escapeshellarg bypass
  - fullwidth characters
---

# Parameter manipulation

When an application spawns a **fixed binary** with an argument array and no shell, metacharacters (`;`, `|`, `$()`) are inert. Argument injection exploits a different seam: the user value lands in `argv` and the **binary itself re-parses it as an option** rather than data. No new process is spawned, yet the existing one is steered into reading files, writing files, or running sub-programs it natively supports.

## Mechanism

```python
# user_arg is attacker-controlled; a fixed URL follows it in argv
subprocess.run(["curl", user_arg, "https://report.internal/collect"], shell=False)
```

No shell, so `;id` does nothing. But a value of `-o/var/www/html/x.php` is parsed by `curl` as the `-o` output flag, so the response from the fixed `https://report.internal/collect` URL is **written** into the web root, a fetch becomes a write. The URL matters: `-o <path>` with no URL in argv just errors with "no URL specified," so this primitive needs a URL present (supplied here by the trailing fixed argument, or by a second flag such as `--url`). The root cause is that **positional data and options share the same argv space**, and most parsers accept options anywhere on the line. Any token beginning with `-` (or `@` for some tools) is a candidate flag.

## Supplying a flag instead of data

The payoff depends on the target binary's option grammar:

```
-o/var/www/html/p.php        # curl: write to web root
-K/etc/passwd                # curl: parse an attacker-named file as a config (one argv element)
--use-askpass=/tmp/x.sh      # wget: command execution
-c core.sshCommand=id        # git: run a command during clone/fetch
--checkpoint-action=exec=id  # tar: command execution
```

Use the `--flag=value` form so the value rides in a single token when the application appends a fixed argument after yours. [GTFOBins](https://gtfobins.github.io/) catalogues these per-binary primitives.

## Worst-fit / fullwidth-character tricks

Some runtimes convert Unicode to ANSI with **best-fit / worst-fit mapping** before handing argv to a native binary. A character that is not a dash can be down-converted into one, slipping a flag past a filter that only blocks ASCII `-`. The classic Windows case maps fullwidth and look-alike codepoints to ASCII:

```
U+FF0D  FULLWIDTH HYPHEN-MINUS   →  -
U+2010  HYPHEN                    →  -
U+FF4F  FULLWIDTH LATIN O         →  o
```

So a value like `？-ｏС:\path` (fullwidth characters) may arrive at the spawned process as `-oC:\path` after worst-fit conversion, re-introducing an option the application believed it had stripped. The same mapping affects quote and space characters, enabling argument splitting where the launcher thought it had passed one token. This is the mechanism behind several `php-cgi`/`CreateProcess` argument-smuggling findings on Windows.

## escapeshellarg / escapeshellcmd bypass concept

PHP's two escapers protect different things, and confusing them leaves a hole:

- `escapeshellcmd()` neutralizes shell metacharacters across the **whole command string** but does **not** quote individual arguments, so it stops a new command yet still lets a value be read as a **flag** (`-o`, `--config`). It is not an argument-injection defense.
- `escapeshellarg()` wraps one argument in quotes to keep it a single positional token. Misuse, escaping the wrong segment, or concatenating an escaped value next to an unescaped `-`, reopens flag injection.

The offensive takeaway: a sink that calls `escapeshellcmd()` (common, because it "looks" like the right function) is frequently still vulnerable to argument injection even though classic metacharacter injection is blocked. Probe with leading-dash values regardless of visible escaping.

## Finding the boundary

The defender's canonical fix is a `--` separator ("everything after this is positional"). If your input is placed **before** any `--`, flags are in play; if after, you are limited to positional abuse, path traversal, `@file` inclusion, or protocol smuggling (`file://`, `gopher://`), rather than flag injection.

## Tools

- **[GTFOBins](https://gtfobins.github.io/)**, per-binary file-read/write and command-exec flags.
- **[Burp Suite](https://portswigger.net/burp)** Repeater/Intruder for fuzzing an input slot with candidate flags.
- Local copies of the target binaries to confirm attached-vs-separate option semantics and `--` handling.

## References

- [CWE-88: Argument Injection](https://cwe.mitre.org/data/definitions/88.html)
- [PortSwigger Research: Argument injection](https://portswigger.net/research)
- [GTFOBins](https://gtfobins.github.io/)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
