---
title: "Instance and SID enumeration in Oracle injection"
description: "Identifying the Oracle instance, SID, and service name from a SQL injection, and how listener information complements it for targeting."
keywords:
  - Oracle SID
  - instance name
  - service name
  - v$instance
  - global_name
---

# Listener and SID enumeration

Knowing the instance identity (SID, service name, global name, host) sharpens later steps and is needed for any direct connection to the database. A SQL injection reads most of it from the data dictionary.

The instance and service identity come from the `V$` views and `SYS_CONTEXT`:

```sql
' UNION SELECT instance_name,host_name,version FROM v$instance-- 
' UNION SELECT SYS_CONTEXT('USERENV','DB_NAME'),SYS_CONTEXT('USERENV','INSTANCE_NAME'),SYS_CONTEXT('USERENV','SERVICE_NAME') FROM dual-- 
' UNION SELECT global_name,NULL,NULL FROM global_name-- 
```

`v$instance` gives the instance name and host, `global_name` the database's global name, and `SYS_CONTEXT` the DB name, instance name, and service name. These identify the SID/service needed to connect a client such as SQLcl or a tool like ODAT straight to the listener.

Beyond the injection, the Oracle TNS listener on TCP 1521 is itself a target: it answers service and version queries, and where the SID or service name is unknown it can be brute-forced against the listener. A legacy, unauthenticated listener may disclose status directly. These are network interactions against the listener rather than SQL injection, but they pair with the in-band identity above: the injection confirms the SID and service name, and the listener is then the route for a direct, higher-bandwidth session once credentials or a hash have been recovered.

## Tools

- **ODAT** (sidguesser module): brute-forces the SID and service name against the TNS listener.
- **Nmap** (`oracle-sid-brute`, `oracle-tns-version` NSE scripts): query the listener on TCP 1521.
- **sqlplus** (or SQLcl): connects directly once the SID and credentials are known.

## References

- Oracle Database Reference: V$INSTANCE, GLOBAL_NAME, SYS_CONTEXT
- OWASP Testing Guide: Testing for SQL Injection
