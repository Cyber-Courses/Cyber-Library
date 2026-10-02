---
title: "OS command injection"
description: "Application code builds an operating-system command from untrusted input, letting an attacker change which program runs or with what arguments, usually remote code execution."
keywords:
  - OS command injection
  - command injection
  - RCE
  - shell injection
  - argument injection
---

# Command

OS command injection occurs when application code builds an operating-system command from untrusted input and hands it to the OS in a way that lets the attacker change **which program runs**, or **with what arguments**. The usual payoff is remote code execution in the context of the service account.

The subtree separates the mechanisms from everything built on top of them: chaining with shell separators, command **substitution**, and **argument** (argv/flag) injection with no shell are the ways in; **exfiltration** covers blind, out-of-band data recovery; and **filter bypass** covers evading blocklists and WAFs.

## Tools

- **[commix](https://github.com/commixproject/commix)**: automated command-injection detection and exploitation.
- **[Burp Suite](https://portswigger.net/burp)**: Repeater and Intruder for injecting and confirming payloads.
- **[interactsh](https://github.com/projectdiscovery/interactsh)**: out-of-band callback capture for blind cases.

## References

- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
