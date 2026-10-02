---
title: "Command execution through IBM Db2 injection"
description: "Reaching OS command execution from a privileged IBM Db2 injection via external C or Java routines, and why ADMIN_CMD and QCMDEXC are not OS shells on Db2 LUW."
keywords:
  - Db2 command execution
  - external routine
  - CREATE PROCEDURE LANGUAGE C
  - ADMIN_CMD
  - QCMDEXC
---

# Command execution

Db2 LUW has no built-in OS-command function, and two names that look like one are commonly misused. `SYSPROC.ADMIN_CMD` runs Db2 CLP administrative commands (such as `EXPORT`, `IMPORT`, `RUNSTATS`, `REORG`), not an arbitrary OS shell. `QSYS2.QCMDEXC` does run OS commands, but it belongs to Db2 for i (IBM i), not Db2 LUW, so it does not apply here. Do not present either as a shell on LUW.

On Db2 LUW the real route is an external routine. With `DBADM` or `CREATE_EXTERNAL_ROUTINE`, define a procedure or function implemented in C or Java that calls out to the operating system, then invoke it:

```sql
CREATE PROCEDURE shell(IN cmd VARCHAR(500)) EXTERNAL NAME 'mylib!run' LANGUAGE C PARAMETER STYLE GENERAL;
CALL shell('id');
```

The external library must exist on the server (in the instance `function` directory), so this usually pairs with a file-write primitive to drop the library first, or reuses a library already present. The routine runs in the Db2 fenced-process account, whose privileges determine the foothold. Java routines (`LANGUAGE JAVA`) that wrap `Runtime.exec()` are the equivalent where Java is configured.

Because defining and running external routines requires high authority and usually a statement/compound context (Db2 does not stack plain statements through standard drivers), command execution is the final step after privilege enumeration confirms the authorities are present. Where they are not, the reachable impact stays at data extraction and the administrative actions `ADMIN_CMD` genuinely allows.

## Tools

- **db2** (or clpplus): the Db2 client for defining and calling the external C or Java routine.
- Manual testing with Burp Repeater and the Db2 client.

## References

- IBM Db2 SQL Reference: CREATE PROCEDURE (external), SYSPROC.ADMIN_CMD
- OWASP Testing Guide: Testing for SQL Injection
