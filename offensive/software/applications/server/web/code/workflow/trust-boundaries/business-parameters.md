---
title: "Business parameter tampering: SKU, plan tier, shipping, and role codes in multi-step requests"
description: "SKU, service tier, region, or role codes carried across steps that the server should resolve from the authenticated identity and inventory service."
keywords:
  - business logic
  - SKU tampering
---

# Business parameters

## Context

Workflows pass productId, plan, or shippingMethod as hidden inputs. If the fulfillment job reads those fields without re-validating ownership and availability, an attacker can substitute a premium SKU at base price.

## Theory

Treat business parameters as hints: always re-fetch authoritative rows by ID on the server before commit.

## Practice

- Complete a purchase for a cheap item; intercept the final POST and swap identifiers for an expensive SKU in a lab catalog.

## Tools

- **Burp Suite**

## References

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
