---
title: "Numeric integrity across checkout and payment: hidden prices, quantities, fees, and stale session totals"
order: 1
description: "Tampering with hidden prices, quantities, or fee lines when the server recomputes totals inconsistently between cart review and payment capture."
keywords:
  - price tampering
  - business logic
---

# Numeric integrity

## Context

The client displays a total; the server stores line items in session or a draft order. If the capture step trusts client-submitted amounts or stale session totals, the paid amount can diverge from catalog prices.

## Theory

Compare every fee and tax field against authoritative catalog and jurisdiction rules on the server at capture time, not only at cart creation.

## Practice

- Change quantity or unit price in a hidden field between review and pay in a staging checkout; diff the charged amount in payment provider logs.

## Tools

- **Burp Suite**

## References

- PortSwigger Web Security Academy: Business logic vulnerabilities
- OWASP WSTG: Testing for Business Logic
