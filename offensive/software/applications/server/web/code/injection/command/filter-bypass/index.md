---
title: "Command injection filter bypass"
description: "Blocklists and WAFs are evaded because the shell normalizes a payload after the filter inspects it — grouped by what each technique defeats."
keywords:
  - filter bypass
  - WAF bypass
  - command injection evasion
  - obfuscation
  - blocklist bypass
---

# Filter bypass

Blocklist and WAF filters invite bypasses because the shell normalizes a payload **after** the filter has inspected it. The techniques group by what they defeat: producing argument separation without spaces (**whitespace**), breaking blocked keywords apart so a literal match fails (**keyword splitting**), rebuilding commands and paths through shell expansion (**expansion and globbing**), and hiding the payload through encoding or casing (**encoding and obfuscation**).
