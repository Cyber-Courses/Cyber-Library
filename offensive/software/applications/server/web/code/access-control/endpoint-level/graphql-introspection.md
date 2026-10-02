---
title: "GraphQL introspection enabled in production: schema and field disclosure to attackers"
description: Production GraphQL servers that answer full schema and type introspection queries, mapping resolvers and argument shapes for follow-on testing.
keywords:
  - GraphQL introspection
  - __schema
  - GraphQL security
  - API discovery
---

# GraphQL introspection

## Context

Introspection (`__schema`, `__type`, field lists) is a first-class GraphQL feature. When it is exposed to low-trust callers in a production deployment, it becomes a complete machine-readable map of mutations, types, and custom scalars. This page is recon and surface mapping, not a substitute for per-field authorization testing.

## Theory

The schema shows which operations exist, including admin or internal-sounding fields. It accelerates BOLA and field-level testing by listing argument names and return types. Hiding introspection reduces casual mapping but does not fix missing authorization on resolvers; the offensive value is speed and completeness of the attack graph in scope.

## Practice

### Run a minimal introspection query in a lab

- POST the standard `__schema` query body to the GraphQL HTTP endpoint with a test token. Save the response JSON for offline navigation of types and fields.

### Diff schema against client bundle

- Compare public schema to mobile or web client queries. Fields that exist only in introspection or in an internal client build are high-priority for missing checks.

## Tools

- **GraphiQL** (in lab only)
- **curl**
- **Insomnia**
