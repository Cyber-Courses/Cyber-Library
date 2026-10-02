---
title: "Swagger UI and OpenAPI docs exposed in production: interactive API exploration risk"
description: Exposed OpenAPI JSON and interactive documentation UIs that enumerate routes, parameters, and auth schemes for a service.
keywords:
  - Swagger UI
  - OpenAPI
  - API documentation
  - openapi.json
  - try it console
---

# Swagger UI

## Context

A reachable `/swagger-ui`, `/openapi.json`, or similar endpoint publishes the contract the server thinks it implements. The offensive value is full path and parameter coverage for planning BOLA, injection, and function-level tests, often including deprecated or internal-tagged operations.

## Theory

Hand-written specs can diverge from code; codegen specs track code more closely. Either way, the file is a recon artifact. Browsers with an active session may issue “try it” requests from the same origin, which matters for how you chain with other issues in a lab; treat the doc endpoint as part of the same site model as the API for session and CSRF experiments when those are in scope.

## Practice

### Fetch the machine-readable spec

- `curl -sS` the `openapi.json` or `swagger.json` URL in scope, or download from the UI’s network tab. Parse paths and methods into a matrix for role testing.

### Tag and version triage

- Read `tags`, `x-internal`, and path `/v1` vs `/v2` blocks. Older version blocks often keep weaker handlers live.

## Tools

- **curl**
- **jq**
- **Burp Suite**
