---
title: "PostgreSQL privilege enumeration via SQL: roles, grants, and superuser detection"
description: Querying pg_roles, information_schema, and session settings to map database privileges after injectable SQL access.
keywords:
  - PostgreSQL privileges
  - SQL injection
---

# Privileges

## Context

Library Structure **Privileges** covers **has_table_privilege**, **pg_roles**, **session_user**, and similar. Useful for **post-exploitation** mapping when SELECT on catalogs is possible through injection.

## See also

- [PostgreSQL (parent)](index.md)
