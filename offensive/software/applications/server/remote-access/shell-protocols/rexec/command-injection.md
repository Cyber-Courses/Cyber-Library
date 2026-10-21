---
title: "Unsafe wrapper construction: command injection through rexec integrations"
order: 1
description: "Command injection is not a rexec protocol flaw. It can occur when a wrapper builds a rexec command from attacker-controlled input and invokes a shell, allowing metacharacters to alter either the local invocation or the remote command string."
keywords:
  - command injection
  - rexec
  - shell metacharacters
  - unsafe construction
  - rce
---

# Unsafe wrapper construction

`rexec` is designed to execute the authenticated user's supplied command, so arbitrary remote command execution with valid credentials is expected behavior rather than command injection. A separate injection vulnerability can arise when a wrapper, web front-end, or management script concatenates attacker-controlled input into a shell command that invokes `rexec`. Depending on where quoting fails, metacharacters may execute locally in the wrapper's context or become part of the remote command string.

```bash
# vulnerable wrapper pattern: attacker-controlled $report_name is concatenated
# into a command interpreted by a shell
run_report() {
  report_name=$1
  sh -c "rexec -l report -p \"$REXEC_PASSWORD\" server 'report $report_name'"
}
# input such as: daily'; id; #
# can terminate the quoted argument and add a local command in the wrapper's context
```

## Exploitation notes

- The flaw is in the caller, not `rexecd`: first establish that attacker input reaches a shell-interpreted wrapper without correct argument separation or escaping.
- Injected commands run as whatever account performs the execution, which for automated management tooling is often privileged.
- This is the same injection class as any unsafe shell-command construction; rexec is simply the sink. Confirm whether a shell interprets the string (metacharacters only help if a shell parses them).
- [Cleartext capture](cleartext-passwords.md) may expose credentials used by the wrapper, but possessing valid credentials and choosing a remote command is normal `rexec` use, not proof of injection.

## References

- [OWASP: command injection](https://owasp.org/www-community/attacks/Command_Injection)
- [HackTricks: rexec](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
