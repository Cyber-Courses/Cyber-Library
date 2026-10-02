---
title: "dict:// URLs in SSRF: minimal dict protocol requests for blind TCP probes and legacy URL handlers"
description: Minimal dict protocol requests used as a blind TCP probe or lightweight SSRF vector on stacks that enable dict URLs.
keywords:
  - SSRF
  - dict protocol
---

# dict:// (SSRF)

## Context

`dict://host:port/` issues a small TCP dialog to dictionary servers. Rare in modern apps but supported in some legacy URL handlers alongside `gopher`. Useful mainly as a **port open** signal or combined with internal hostnames in CTF-style stacks.

## Theory

Same allowlist bypass pattern as gopher: scheme not considered in `http`-only regex filters.

## Practice

- Probe whether `dict://127.0.0.1:11211/` returns distinguishable errors vs closed port in a test app.

## Tools

- **Burp Suite Collaborator** (timing and DNS)