---
title: "Stored HTML and server-side sanitization gaps: allowlists, parser differentials, and mutation XSS"
description: Rich text stored and rendered through HTML cleaners where configuration, nested tags, or downstream parsers re-open XSS when the primary sink is server-side.
keywords:
  - XSS
  - HTML sanitization
---

# Stored HTML sanitization

## Context

**Server-side** cleaners (allowlist tags, strip scripts) can still fail on **nested** **foreign content**, **SVG/MathML**, or **parser** differences between **sanitize** time and **browser** parse time. This page is about **application** choice of sanitizer config, not only client frameworks.

## Theory

Treat sanitization as **versioned** configuration; fuzz with **mutation** payloads appropriate to the allowed tag set.

## See also

- [Markup injection (parent)](index.md)
