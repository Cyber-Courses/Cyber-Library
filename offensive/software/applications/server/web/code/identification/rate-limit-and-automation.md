---
title: "Rate limits, throttling, and automation surfaces for credential attacks"
description: Per-IP-only limits, client-device gaps, and API keys that enable credential stuffing or OTP brute force at application layer.
keywords:
  - credential stuffing
  - rate limiting
  - brute force
---

# Rate limits and automation

Weak throttling (**per IP** only, **no** account lockout coordination, **large** OTP space with **fast** resend) shapes whether credential stuffing or **OTP guessing** is practical. Mobile and web clients may hit **different** endpoints with **different** limits.
