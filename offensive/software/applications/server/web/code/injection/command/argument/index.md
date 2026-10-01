---
title: "Argument injection"
description: "A user value lands in argv and the called binary re-parses it as an option rather than data — no shell, no new process, yet file read/write and code execution."
keywords:
  - argument injection
  - argv injection
  - option injection
  - flag injection
  - parameter manipulation
---

# Argument

Argument injection targets the no-shell case: a fixed binary is spawned with an argument array, so shell metacharacters are inert — but a user-controlled value lands in `argv` where the program re-parses it as an **option** instead of data. No new process is spawned, yet the existing one is steered into reading files, writing files, or launching helper programs it natively supports (`curl -o`, `git -c`, `tar --checkpoint-action`, and so on).
