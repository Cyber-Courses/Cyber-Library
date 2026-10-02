---
title: "IDOR and BOLA: direct reference tampering when servers skip per-object authorization"
description: Manipulating object identifiers in paths, query strings, or JSON bodies to access other users' records when the server omits a per-object authorization check.
keywords:
  - IDOR
  - BOLA
  - object level authorization
  - insecure direct object reference
---

# Direct reference (IDOR)

## Context

The server loads a row by an id, slug, or composite key from the request. The subject is authenticated, but the code does not assert that the subject may access that specific row. Swapping one id for another in a lab with two test accounts is the standard controlled proof. This is not “guessing random UUIDs” as a complete story: the bug is a missing check, not key length.

## Theory

Single-table `SELECT` by primary key with no `owner_id` or `tenant_id` in the `WHERE` clause, list UIs that filter the list but not the detail route, and GraphQL `node(id:)` resolvers that resolve any global id are the same class. Batch and nested-resource variants extend the id set in one request or one path.

## Practice

### Horizontal test with two accounts

- As user A, create or read an object and note its id. As user B, call the same read or write route with A’s id in the path or body. A `200` with A’s data while authenticated as B is a direct BOLA signal in the lab.

### Fuzz id type and encoding

- Try integer `+1` neighbors, different string encodings of the same id, and parallel array fields when the API accepts `ids[]` or a JSON list of references.

## Tools

- **Burp Suite**
- **curl**
- **Autorize** (Burp extension)
