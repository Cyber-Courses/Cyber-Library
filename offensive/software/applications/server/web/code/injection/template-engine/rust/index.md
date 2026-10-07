---
title: "Rust server-side template injection"
order: 8
description: "SSTI in Rust template engines: Tera's runtime-parsed expressions, why compile-time engines are not injectable, and the information-disclosure ceiling Tera's sandbox imposes."
keywords:
  - Rust SSTI
  - Tera
  - template injection
  - compile-time templates
---

# Rust

Most Rust template engines (Askama, Maud, Yarte) compile templates at build time from files in the source tree, so there is no runtime sink that parses attacker-controlled template text, and they are not injectable. The case that matters for SSTI is Tera, a Jinja2-inspired engine that parses templates at runtime and is therefore reachable when an application calls `Tera::one_off` or renders a template string built from user input.

Tera is another honest, limited case: like Go, it evaluates expressions and calls registered filters and functions but exposes no path to the Rust runtime, so impact is normally information disclosure rather than RCE.

## Engines

- **[Tera](tera.md)**: `{{7*7}}` confirmation, context and filter enumeration, and the limits that keep it to disclosure.

## References

- Tera documentation (Keats/tera)
- PortSwigger Web Security Academy: Server-side template injection
