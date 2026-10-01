---
title: "Spaceless and whitespace bypass"
description: "Defeating space- and line-based filters with ${IFS}, brace expansion, and newline or backslash-newline continuations."
keywords:
  - spaceless
  - IFS
  - no space
  - newline injection
  - line continuation
---

# Whitespace

Techniques that defeat filters targeting the space character or line structure: `${IFS}`, brace expansion, and input redirection separate arguments without a literal space, while newlines and backslash-newline continuations split or terminate a command inside a spawned shell.
