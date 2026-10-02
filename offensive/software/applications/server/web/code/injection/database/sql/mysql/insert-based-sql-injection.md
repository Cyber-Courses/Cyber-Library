---
title: "INSERT-based SQL injection in MySQL: ON DUPLICATE KEY UPDATE and second-order inserts"
description: Injecting into INSERT paths that later affect authentication or admin rows.
keywords:
  - INSERT injection
  - MySQL SQL injection
---

# INSERT-based SQL injection

## Context

Library Structure **INSERT** topic, **ON DUPLICATE KEY UPDATE** can change **password** hashes when unique keys collide. Requires injectable **INSERT** surface.
