---
title: "Port scanning via HTTP SSRF: inferring open internal services from errors, timing, and response leaks"
description: Using an http:// URL to an internal host with varying ports to infer open services from errors, timing, or banner snippets.
keywords:
  - SSRF
  - port scan
---

# Port scanning (HTTP SSRF)

## Context

When the SSRF sink returns **full or partial** response bodies, or **distinct** errors per port (connection refused vs timeout vs TLS alert), you can sweep `http://internal:PORT/` through the vulnerable parameter. Respect program rules: many assessments forbid **aggressive** scanning.

## Theory

Blind mode relies on timing and TCP reset patterns; verbose mode may leak SSH banners, Redis errors, or HTTP status fragments in the application’s forwarded or reflected output.

## Practice

- In scope, step through a **small** port list on **one** internal IP with **rate** limits; log status code, length, and substring signatures.

## Tools

- **ffuf** (against your lab)
- **Burp Suite**