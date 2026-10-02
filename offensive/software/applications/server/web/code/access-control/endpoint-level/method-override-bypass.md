---
title: "HTTP method override bypass: X-HTTP-Method-Override and verbs through restrictive gateways"
description: Frameworks that honor X-HTTP-Method-Override or _method= allowing POST to impersonate DELETE or PUT past naive gateway rules.
keywords:
  - X-HTTP-Method-Override
  - method tunneling
  - verb bypass
  - REST authorization
---

# Method override

## Context

Some stacks map an incoming `POST` to `DELETE` or `PUT` when a header or form field says so, for clients that cannot emit certain verbs. If routing or authorization is keyed on the * wire* verb as seen by the outer hop but the inner app rewrites the effective verb, a route that was “POST-only” in the gateway’s mind can still execute a state-changing method on the app.

## Theory

The mismatch is between the method the edge policy sees and the method the route matcher uses after override. The bypass appears when one layer uses `POST` and another applies `DELETE` with the same path and body.

## Practice

### Send POST with override header in a lab

- Issue `POST` to a path that returns `405` for `DELETE` at the edge, with `X-HTTP-Method-Override: DELETE` and the same body shape a real `DELETE` would use, and observe the application’s status and side effects on a test object.

## Tools

- **curl**
- **Burp Suite**
