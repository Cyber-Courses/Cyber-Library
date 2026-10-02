---
title: "NoSQL injection"
description: "Injection against non-relational data stores, where attacker-controlled objects, operators, or query-language fragments are interpreted by the driver rather than treated as data."
keywords:
  - NoSQL injection
  - MongoDB
  - Cassandra
  - Redis
  - operator injection
  - Cypher injection
---

# NoSQL

NoSQL injection adapts the injection idea to non-relational data stores, where the dangerous primitive is not always a query string.

It spans operator and syntax injection in document stores (MongoDB), query-language injection in wide-column (Cassandra CQL) and graph (Neo4j Cypher) databases, search-DSL injection (Elasticsearch), command and scripting injection in key-value stores (Redis and CouchDB views), and expression injection in managed stores (DynamoDB). Because the techniques differ sharply per engine, this subtree is organized by database. A common thread runs through many of them: an attacker-controlled object or operator that the driver interprets rather than treating as plain data, often reachable when request JSON is passed straight into a query.

## Tools

- **NoSQLMap**: automated detection and exploitation across several NoSQL engines.
- **nosqli**: Go-based NoSQL injection scanner for web endpoints.
- **Burp Suite**: intercept and tamper with database-backed requests.

## References

- [OWASP Web Security Testing Guide: Testing for NoSQL Injection](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
