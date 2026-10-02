---
title: "MySQL WAF bypass: wide-byte and GBK charset escape confusion"
description: Multibyte character sets where quote bytes recombine after escape—legacy PHP/MySQL stacks; test only in lab replicas.
keywords:
  - WAF bypass
  - GBK
  - wide byte
---

# Wide byte / GBK

## Context

Library Structure **Wide Byte Injection GBK**: `SET NAMES gbk` and `%bf%27` style sequences that consume the backslash from `addslashes`. **Rare** in modern UTF-8 stacks but documented for **historical** audits.

## See also

- [WAF bypass (parent)](index.md)
