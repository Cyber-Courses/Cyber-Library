---
title: "Oracle DBMS_SCHEDULER jobs and external execution risk"
description: Scheduler jobs as a persistence and execution path, create_job and external job types under strict operational control.
keywords:
  - DBMS_SCHEDULER.create_job
  - external job
---
# DBMS SCHEDULER jobs (Oracle)

**DBMS_SCHEDULER** can create **jobs** that run **PL/SQL** or, in some configurations, **external** actions. Combined with **weak ACLs**, this is a **high** impact surface.

## Context

`DBMS_SCHEDULER.CREATE_JOB` schedules PL/SQL or external jobs, powerful persistence and execution.
## Technique

External job types need credential objects; PL/SQL jobs run as job owner.
## Practice

- List `DBA_SCHEDULER_JOBS` for hijackable weak jobs.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.
