---
title: "MongoDB operator injection: $ne, $gt, $regex, and JSON query shapes"
description: Attacker-controlled BSON/JSON objects passed to find() or aggregation that use operators to bypass authentication filters.
keywords:
  - NoSQL injection
  - MongoDB
---

# MongoDB operators

## Context

**Login** lookups that match `{"username": user, "password": pass}` fail open when `user` is replaced with **`{"$ne": null}`** and operators are not stripped. **$regex** enables blind extraction.
