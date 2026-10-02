---
title: "Account enumeration via application signals: errors, timing, and registration flows"
description: Distinct HTTP responses, timing, or workflow branches that reveal whether an identifier exists before authentication succeeds.
keywords:
  - account enumeration
  - user enumeration
---

# Account enumeration

Applications leak **existence** of accounts through **different** error strings (“unknown user” vs “bad password”), **different** HTTP status or field errors on password reset, **branching** in OAuth or magic-link flows, or **consistent** timing differences when crypto work differs.

## Practice

- In authorized tests, compare responses for known-valid vs known-invalid identifiers across login, reset, and registration with **identical** request shapes where possible.

## See also

- [Identification (parent)](index.md)
- [Authentication](../authentication/index.md)
