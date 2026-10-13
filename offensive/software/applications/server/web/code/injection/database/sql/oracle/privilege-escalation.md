---
title: "Privilege escalation in Oracle injection"
order: 10
description: "Escalating from a low-privileged Oracle schema to DBA through definer-rights package injection, dangerous system privileges, and role abuse."
keywords:
  - Oracle privilege escalation
  - GRANT DBA
  - CREATE ANY PROCEDURE
  - definer rights
  - CREATE ANY JOB
---

# Privilege escalation

A low-privileged Oracle account rarely stops an attacker, because Oracle ships many routes from an ordinary schema to DBA. The target is almost always to run `GRANT DBA TO <me>` (or `ALTER USER`) in a context that has permission to do so.

The classic route is injecting a definer-rights procedure owned by a privileged schema. A vulnerable `SYS`-owned package that builds dynamic SQL from input runs the injected statement as `SYS`, so a payload of `GRANT DBA TO SCOTT` succeeds. Over the years a long list of supplied packages has been exploitable this way, which is why patching and least privilege on PL/SQL matter.

Dangerous system privileges held directly are the other route. `CREATE ANY PROCEDURE` lets you create a definer-rights procedure in a privileged schema, but creating it is not enough on its own: you also need a way to invoke it as that owner, through `EXECUTE ANY PROCEDURE`, an explicit execute grant, or a gadget that runs it (a scheduler job or a trigger the owner fires). `CREATE ANY TRIGGER` lets you plant a trigger that fires as a privileged owner, which is itself such an invocation gadget. `CREATE ANY JOB` or `CREATE EXTERNAL JOB` reaches OS command execution through `DBMS_SCHEDULER`. `EXECUTE ANY PROCEDURE` plus a vulnerable package combines into escalation. Enumerate what the session holds first:

```sql
' UNION SELECT privilege,NULL,NULL FROM session_privs-- 
```

Java permissions are a further path: a schema granted `java.io.FilePermission` or `java.lang.RuntimePermission` can load and run code with the database server's OS privileges. The pattern throughout is to find one over-granted privilege or one injectable privileged procedure, then use it to grant yourself `DBA`, after which command execution, file access, and persistence all open up.

## Tools

- **ODAT** (Oracle Database Attacking Tool): tests and abuses dangerous privileges and definer-rights packages.
- **sqlplus** (or SQLcl): the Oracle client for running `GRANT` and reading `session_privs`.
- Manual testing with Burp Repeater and the Oracle client.

## References

- Oracle Database Security Guide: system privileges, definer's rights
- OWASP Testing Guide: Testing for SQL Injection
