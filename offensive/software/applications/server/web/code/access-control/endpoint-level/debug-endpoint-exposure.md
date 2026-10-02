---
title: "Debug and actuator endpoints: stack traces, env dumps, and route listings in production"
description: Development or diagnostic HTTP routes still registered in production builds, often returning stack traces or control-plane actions.
keywords:
  - debug route
  - test endpoint
  - actuator
  - development only API
---

# Debug endpoints

## Context

Debug and actuator-style routes often ship with a framework and are meant to be disabled or network-restricted in production. When they stay bound and reachable, they return environment details, configuration, or one-click actions (cache clear, feature toggle) that normal users should not trigger.

## Theory

Fingerprinting uses predictable paths (`/actuator`, `/debug`, framework defaults), distinct error bodies on `GET` vs `POST`, and banner strings in JSON error envelopes. The impact is information and sometimes state change, not a novel HTTP primitive.

## Practice

### Path and verb discovery in scope

- In a test environment, `GET` and `POST` common default paths with a small wordlist derived from the stack’s documentation. Record status codes and content types; `200` with a JSON `beans` or `env` object flags classic actuator exposure when the product is Java/Spring-flavored.

## Tools

- **ffuf**
- **curl**
- **Burp Suite**
