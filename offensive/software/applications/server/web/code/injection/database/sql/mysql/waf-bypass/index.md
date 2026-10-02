---
title: "MySQL SQL injection WAF bypass: information_schema alternatives, comments, encodings"
description: Obfuscation and metadata alternatives aligned with Library Structure MySQL WAF Bypass subtree.
keywords:
  - WAF bypass
  - MySQL SQL injection
---

# WAF bypass (MySQL)

## Pages

| Page | Focus |
|------|--------|
| [Alternative to information_schema](alternative-to-information-schema.md) | innodb_table_stats, SHOW |
| [Alternative to VERSION](alternative-to-version.md) | @@innodb_version, globals |
| [Alternative to GROUP_CONCAT](alternative-to-group-concat.md) | JSON_ARRAYAGG, CONCAT_WS |
| [Conditional comments](conditional-comments.md) | /*!version*/ execution |
| [Scientific notation](scientific-notation.md) | 1e1 numeric tricks |
| [Wide byte / GBK](wide-byte-gbk.md) | Multibyte escape confusion |
