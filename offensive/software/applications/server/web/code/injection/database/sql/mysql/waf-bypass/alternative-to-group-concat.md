---
title: "MySQL WAF bypass: alternatives to GROUP_CONCAT (JSON_ARRAYAGG, CONCAT_WS)"
description: Concatenation limits and WAF rules on GROUP_CONCAT, staging-only payload research.
keywords:
  - WAF bypass
  - MySQL SQL injection
---

# Alternative to GROUP_CONCAT

## Context

Library Structure **Alternative to GROUP CONCAT**: **JSON_ARRAYAGG**, **CONCAT_WS**, or chunked queries when **GROUP_CONCAT** hits length or filter limits.
