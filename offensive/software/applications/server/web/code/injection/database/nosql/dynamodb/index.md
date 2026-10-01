---
title: "DynamoDB injection"
description: "Amazon DynamoDB is schemaless and key-value, but the APIs that query it (PartiQL statements and filter/condition expressions) are still injectable when built from untrusted input."
keywords:
  - DynamoDB injection
  - PartiQL injection
  - filter expression injection
  - condition expression
  - NoSQL injection
  - AWS
---

# DynamoDB

Amazon DynamoDB has no SQL engine in the classic sense, so it is often assumed immune to injection. It is not. DynamoDB exposes two query surfaces that application code routinely builds from user input, and both are injectable when that input is concatenated into a statement or expression string rather than passed as parameters.

The first surface is **PartiQL**, a SQL-compatible query language run through `ExecuteStatement`, `BatchExecuteStatement`, and the transaction APIs. Code that string-builds a PartiQL statement reopens the full class of statement injection: breaking out of a value, widening a `WHERE` with `OR`, and reading items the caller was never scoped to.

The second surface is the **expression** family: `FilterExpression`, `KeyConditionExpression`, and `ConditionExpression`, together with the `ExpressionAttributeNames`/`ExpressionAttributeValues` maps that feed them. When attacker input shapes the expression text or those maps, a filter can be widened to return more items, or a conditional write guard can be made to pass when it should fail.

