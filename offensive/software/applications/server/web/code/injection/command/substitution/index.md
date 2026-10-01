---
title: "Command substitution"
description: "Backticks and $(...) run a nested command and splice its output into the current command line, even mid-argument."
keywords:
  - command substitution
  - subshell
  - backticks
  - dollar parentheses
  - nested command
---

# Substitution

Command substitution — `` `cmd` `` and `$(cmd)` — runs a nested command and splices its output into the surrounding command line. Because it is evaluated even when the injection point sits in the middle of an argument, it is both a primary execution primitive and one of the most reliable confirmation oracles for command injection.
