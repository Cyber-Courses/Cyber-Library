---
title: "MySQL read file via LOAD_FILE in SQL injection"
description: Reading local files when FILE privilege and secure_file_priv allow, error/union channels.
keywords:
  - LOAD_FILE
  - MySQL SQL injection
---

# Read file (LOAD_FILE)

## Context

Library Structure **Read Content of a File**, **LOAD_FILE** in **UNION** or **error** slots. Blocked by **secure_file_priv** on hardened servers.
