---
title: "Property type confusion: unexpected JSON shapes, file uploads, and parser coercion"
description: Coercing a field to an array, object, or alternate JSON type to bypass a schema that only validated a string form.
keywords:
  - type confusion
  - JSON type coercion
  - validation bypass
---

# Property type confusion

## Context

Validation often runs on a DTO that expects a string for `id`. If the actual parser first coerces `["1","2"]` or an object to a string, or the ORM maps types differently, the check and the sink see different shapes.

## Theory

NoSQL query builders that accept `{"$gt":""}` style operator objects are the classic type confusion in another layer; in JSON binding, the same idea is `role` as list vs string. The test is to vary JSON types for the same key.

## Practice

### Send string vs array vs object for one field

- In a lab, fix all other required fields, then try `"role": "user"`, `"role": ["admin"]`, and `{"role":{"name":"admin"}}` on the same update route. Record which forms pass validation and what is stored.

## Tools

- **curl**
- **Burp Suite**
