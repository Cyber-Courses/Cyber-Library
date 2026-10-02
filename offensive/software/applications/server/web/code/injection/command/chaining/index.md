---
title: "Command chaining and separators"
description: "Shell control operators that let an injected value terminate the intended command and start another: ; && || and &."
keywords:
  - command chaining
  - command separator
  - sequential execution
  - conditional execution
  - background execution
---

# Chaining

Shell control operators let an injected value end the intended command and begin another. A semicolon runs commands **sequentially**; `&&` and `||` run them **conditionally** on the previous command's success or failure (a useful boolean oracle); and a trailing `&` runs a job in the **background**. Whenever the application invokes a shell, any of these turns one injectable parameter into arbitrary command execution.

## Tools

- **[commix](https://github.com/commixproject/commix)**: automated command-injection detection and exploitation.
- **[Burp Suite](https://portswigger.net/burp)**: Repeater and Intruder for separator injection.

## References

- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
