---
title: "Endpoint-level access control: routes, admin surfaces, GraphQL, and hidden functions"
description: Authorization flaws where an HTTP route, RPC method, or internal function is callable without the correct role, scope, or tenant check.
keywords:
  - function level authorization
  - endpoint authorization
  - missing role check
  - horizontal privilege escalation
  - vertical privilege escalation
---

# Endpoint-level access control

**Function-level** (endpoint) authorization governs *which* operations a caller may use: admin APIs, internal tools exposed over HTTP, debug routes, and GraphQL fields or resolvers. The application may authenticate the user and still be wrong if **authorization** for the **action** is missing, inconsistent, or only enforced on the client.

## Pages

- [Missing role check](missing-role-check.md)
- [GraphQL introspection](graphql-introspection.md)
- [Swagger UI and OpenAPI surface](swagger-ui.md)
- [HTTP method override bypass](method-override-bypass.md)
- [Debug endpoint exposure](debug-endpoint-exposure.md)
- [Admin panels](admin-panels.md)
- [Test resources in production](test-resources.md)
- [Hidden admin function](hidden-admin-function.md)
