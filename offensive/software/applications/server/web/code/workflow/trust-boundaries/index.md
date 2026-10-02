---
title: "Trust boundaries in web workflows: tampering with prices, SKUs, tiers, and cross-step business parameters"
description: Numeric and business parameters that cross steps—amounts, quantities, roles—must stay consistent with server-side policy.
keywords:
  - business logic
  - parameter tampering
---

# Trust boundaries

Trust boundaries are the **edges** where data leaves one enforcement context and enters another: from browser to API, from microservice A to B, or from synchronous form to async job. The library focuses on **numeric** and **business** parameters that are easy to tamper when hidden fields or JWT claims are trusted blindly.

## Pages

| Page | Focus |
|------|--------|
| [Numeric integrity](numeric-integrity.md) | Prices, fees, tax lines, quantities |
| [Business parameters](business-parameters.md) | Product SKU, shipping tier, entitlement tier |

## See also

- [Workflow (parent)](index.md)
- [Access control](../access-control/index.md)
- [Parallel effects](parallel-effects/index.md)
