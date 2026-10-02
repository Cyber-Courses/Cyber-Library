---
title: "Blind SQL injection (MySQL): boolean, substring, REGEXP, and timing channels without visible errors"
description: Boolean-based blind SQLi in MySQL using conditional expressions, LIKE, REGEXP, MAKE_SET, and substring tests when no union or error channel exists.
keywords:
  - blind SQL injection
  - boolean SQLi
  - MySQL
---

# Blind SQLi (MySQL)

Blind SQL injection returns no direct query output: you infer true vs false from response differences (length, status, body hash) or secondary signals. MySQL provides `IF`, `LIKE`, `REGEXP`, `SUBSTRING`, `MAKE_SET`, and substring equivalence patterns to extract data bit-by-bit or character-by-character in authorized labs only.

## Pages

- [Conditional statement](conditional-statement.md)
- [LIKE](like.md)
- [MAKE SET](make-set.md)
- [REGEXP](regexp.md)
- [Substring equivalent](substring-equivalent.md)

## See also

- [MySQL (parent)](../index.md)
