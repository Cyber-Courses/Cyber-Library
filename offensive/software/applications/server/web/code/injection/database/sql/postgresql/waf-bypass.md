---
title: "PostgreSQL SQL injection WAF bypass: dollar quoting, CHR concatenation, and comments"
description: Obfuscation and dialect tricks for PostgreSQL SQLi when a WAF or filter sits in front of the application, operator tradecraft, not a substitute for bind parameters.
keywords:
  - WAF bypass
  - PostgreSQL SQL injection
---

# WAF bypass

## Context

Maps to Library Structure **WAF Bypass** for PostgreSQL: **$$** dollar quoting, **CHR()** chains, **--** / **/***\***/ comments, and **Unicode** tricks. Treat as **test-plan** notes for **your** WAF rules in staging.
