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

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess.

OS command injection occurs when application code builds an operating-system command from untrusted input and hands it to the OS in a way that lets the attacker change **which program runs**, or **with what arguments**. The usual payoff is remote code execution in the context of the service account.

The subtree separates the mechanisms from everything built on top of them: chaining with shell separators, command **substitution**, and **argument** (argv/flag) injection with no shell are the ways in; **exfiltration** covers blind, out-of-band data recovery; and **filter bypass** covers evading blocklists and WAFs.
