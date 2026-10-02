---
title: "Persistence techniques in Oracle after injection"
description: "Maintaining access to an Oracle database after a privileged injection using scheduler jobs, triggers, and backdoor procedures and accounts."
keywords:
  - Oracle persistence
  - DBMS_SCHEDULER job
  - trigger backdoor
  - stored procedure backdoor
  - persistence through grants
---

# Persistence

Once an injection has reached DBA-level control, persistence keeps that access across password changes and patches. These all require the high privilege obtained through escalation, and run from an injectable PL/SQL context.

A scheduler job re-establishes access on a timer. A `DBMS_SCHEDULER` job that re-grants a role or runs an external program fires on its schedule regardless of later credential resets:

```sql
BEGIN DBMS_SCHEDULER.CREATE_JOB(job_name=>'sync$',job_type=>'PLSQL_BLOCK',job_action=>'BEGIN EXECUTE IMMEDIATE ''GRANT DBA TO lowpriv''; END;',repeat_interval=>'FREQ=HOURLY',enabled=>TRUE); END;
```

A trigger plants code that runs on an event. A logon or DML trigger owned by a privileged schema, or one that re-adds a dropped privilege, survives until noticed. A backdoor stored procedure with definer rights in a privileged schema gives on-demand escalation: calling it re-grants DBA or runs a command.

A quieter option is a dormant account or grant: creating a low-profile user with a role grant, or adding the attacker's schema to a role that is rarely audited, leaves a credential to return with. Naming objects to blend with Oracle's own (`SYS`-style names, `$` suffixes) delays discovery.

Each of these is a post-compromise action that assumes DBA already, so it follows privilege escalation and command execution rather than standing alone. The value is durability: a single job or trigger re-creates the access an incident responder removes elsewhere.

## References

- Oracle Database PL/SQL Packages and Types Reference: DBMS_SCHEDULER; SQL Language Reference: CREATE TRIGGER
- OWASP Testing Guide: Testing for SQL Injection
