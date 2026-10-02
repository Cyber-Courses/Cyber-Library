---
title: "Django ORM injection"
description: "Injection through Django's raw(), extra(), the low-level cursor, and user-controlled filter() lookups."
keywords:
  - Django ORM
  - raw()
  - extra()
  - filter() injection
  - Python
---

# Django ORM

Django's ORM parameterizes normal querysets, but several APIs reopen injection when built from untrusted input: `raw()` and the low-level `connection.cursor()`, the legacy `extra()` with its `select`/`where` fragments, and user-controlled field or lookup names passed into `filter()`.
