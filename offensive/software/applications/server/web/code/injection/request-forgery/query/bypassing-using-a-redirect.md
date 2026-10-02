---
title: "SSRF via HTTP redirects: open redirect chains, allowlisted hosts, and internal landing URLs"
description: Chaining open redirects so a server-side fetch to an allowed host ends on an internal or metadata URL after HTTP 3xx.
keywords:
  - SSRF
  - open redirect
  - redirect chain
---

# Redirect chains (SSRF)

## Context

Allowlists often approve a partner hostname. If that host issues `302` to `http://169.254.169.254/` (or any internal address), the vulnerable server may follow the redirect while filters only checked the first URL. Reproduce only in labs with a redirector you control and written permission.

## Theory

Clients differ: max redirect hops, whether redirects are followed for POST, and whether allowlists re-run per hop. Map the exact HTTP client class in the application.

## Practice

- In a test harness, point the SSRF sink at your redirector URL; have it redirect to an in-lab metadata mock or black-hole internal IP and compare response bodies and timing to a direct blocked URL.

## Tools

- **Burp Suite Collaborator**
- **curl** with and without `-L`