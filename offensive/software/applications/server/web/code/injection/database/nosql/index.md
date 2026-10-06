---
title: "NoSQL injection"
order: 1
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

## Subtopics

- **[Cassandra](cassandra/index.md)**: String-built CQL against Apache Cassandra lets an attacker break out of quoted values, add predicates, force cluster-wide scans, and smuggle extra writes int...
- **[CouchDB](couchdb/index.md)**: Apache CouchDB is an HTTP/JSON document database whose design documents run server-side JavaScript, so injection targets those functions and the query, view,...
- **[DynamoDB](dynamodb/index.md)**: Amazon DynamoDB is schemaless and key-value, but the APIs that query it (PartiQL statements and filter/condition expressions) are still injectable when built...
- **[Elasticsearch](elasticsearch/index.md)**: User input reaching Lucene query_string syntax, the JSON Query DSL body, or Painless scripts reopens injection on Elasticsearch and OpenSearch clusters.
- **[MongoDB](mongodb/index.md)**: Operator and syntax injection against MongoDB, where request JSON reaches a query unsanitized and attacker-controlled objects are interpreted as query operat...
- **[Neo4j](neo4j/index.md)**: String-built Cypher against Neo4j graph databases lets an attacker alter MATCH patterns, cross label boundaries, abuse APOC, and infer data blindly.
- **[Redis](redis/index.md)**: Command injection via the RESP protocol and abuse of dangerous commands in a Redis key-value store, from CRLF smuggling to CONFIG SET RCE and server-side Lua.

## Tools

- **NoSQLMap**: automated detection and exploitation across several NoSQL engines.
- **nosqli**: Go-based NoSQL injection scanner for web endpoints.
- **Burp Suite**: intercept and tamper with database-backed requests.

## References

- [OWASP Web Security Testing Guide: Testing for NoSQL Injection](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
