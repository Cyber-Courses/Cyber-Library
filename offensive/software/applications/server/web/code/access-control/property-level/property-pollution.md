---
title: "Property pollution: prototype pollution and deep-merge abuse in JavaScript stacks"
description: Duplicate keys, type coercion, and array forms that make the server bind a different effective value than the client UI sent.
keywords:
  - parameter pollution
  - duplicate key
  - HPP
---

# Property pollution

## Context

HTTP query and form parsers, and some JSON parsers, pick first or last duplicate keys, or coerce `id=1` and `id[]=2` into different types. A policy that blocks `user_id` at the first pass may not see the second key.

## Theory

The class sits between delivery (how bytes arrived) and binding (how the object is built). The offensive test is always: send the same logical field twice in every shape the stack accepts for that content type.

## Practice

### Duplicate the same key in query and in JSON body

- In a lab, set `?role=user` and POST JSON `{"role":"admin"}` if the handler merges query and body. Observe which wins in the built object in error messages or round-trip `GET`.

## Tools

- **Burp Suite**
- **curl**
