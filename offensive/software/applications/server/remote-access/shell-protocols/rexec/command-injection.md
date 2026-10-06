---
title: "Command injection: abusing unsafe rexec command construction"
order: 1
description: "rexec executes a command string on the server, and where that string is built from attacker-influenced input by a wrapper, script, or application that calls rexec unsafely, shell metacharacters inject additional commands. This extends a single intended command into arbitrary execution as the account running it."
keywords:
  - command injection
  - rexec
  - shell metacharacters
  - unsafe construction
  - rce
---

# Command injection

rexec runs a command string on the target, and command injection arises not in the protocol itself but wherever something builds that string from attacker-influenced input without sanitisation. Management scripts, web front-ends, and applications that shell out to rexec (or to the server-side command) by concatenating user input are the vulnerable layer: injecting shell metacharacters (`;`, `|`, `` ` ``, `$()`) into a parameter that lands in the executed command appends attacker commands, which run as the account executing them. This turns one intended operation into arbitrary command execution.

```bash
# where an app/script passes user input into a rexec'd command unsafely:
#   intended:  rexec ... host "report <USERINPUT>"
#   injected:  USERINPUT = "x; id; cat /etc/passwd"
rexec -l user -p pass <target> 'report x; id; cat /etc/passwd'
# or server-side, where rexecd runs a command that itself concatenates attacker data
#   supply metacharacters in the field that reaches the shell
```

## Exploitation notes

- The flaw is in the caller, not rexecd: look for scripts, cron jobs, or applications that construct a rexec (or server-side) command from parameters you control, and inject shell metacharacters into those parameters.
- Injected commands run as whatever account performs the execution, which for automated management tooling is often privileged.
- This is the same injection class as any unsafe shell-command construction; rexec is simply the sink. Confirm whether a shell interprets the string (metacharacters only help if a shell parses them).
- Combine with [cleartext capture](cleartext-passwords.md) (to get the credential the wrapper uses) or [trust](trusted-host-bypass.md) to reach the injectable path.

## References

- [OWASP: command injection](https://owasp.org/www-community/attacks/Command_Injection)
- [HackTricks: rexec](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
