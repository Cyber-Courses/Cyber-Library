---
title: "Test and staging endpoints in production: sample data, mock payments, and debug routes"
description: Staging routes, test accounts, and sample data left in live deployments, often with weaker access rules than production features.
keywords:
  - test accounts
  - staging data
  - default credentials
  - demo mode
---

# Test resources

## Context

“Test” and “demo” features are often implemented with relaxed rate limits, known passwords, or extra debug fields. They can share the same code path as production features with a feature flag, which makes them high-yield in scope when the flag is wrong in prod.

## Theory

Discovery comes from static strings in client JS, `robots.txt`, DNS names (`staging.example.com` pointing to prod-like infra), and error messages that name test tenants. The abuse is usually the same BOLA or function-level class with a smaller guard surface.

## Practice

### Search client artifacts for test hostnames and flags

- Extract strings from mobile and web bundles for `test`, `demo`, `internal` subdomains, then try those hostnames and path prefixes on the in-scope program with the same test accounts you already have.

## Tools

- **strings**
- **Burp Suite**
- **curl**
