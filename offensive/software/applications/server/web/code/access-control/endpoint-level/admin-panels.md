---
title: "Admin panel exposure: default URLs, weak credentials, and leaked management UIs"
description: Management UIs and consoles exposed on common paths or protected only by default credentials, enabling high-impact state changes in scope.
keywords:
  - admin panel
  - management console
  - default credentials
  - web console
---

# Admin panels

## Context

Admin panels collect user management, configuration, and data export in a browser UI. Offensive value is high when the path is guessable, the install still uses default credentials, or the panel is exposed to the open internet while the product assumed a management VLAN.

## Theory

Typical paths include `/admin`, `/wp-admin`, product-specific console URLs, and database admin UIs left under the app root. Success is often a combination of path discovery, default password lists, and session behavior after login (cookie scope, lack of MFA on the panel only).

## Practice

### Map panel auth model in a lab

- After login, identify whether the session is the same as the main app or a separate cookie name. Test whether a low-privilege app session can `GET` admin paths without a second factor when the product claims MFA is on.

## Tools

- **Burp Suite**
- **browser**
