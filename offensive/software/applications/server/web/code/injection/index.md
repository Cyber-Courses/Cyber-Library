---
title: "Injection in web applications: SQL, command, SSRF, templates, and unsafe parsing sinks"
description: "Where untrusted data becomes part of what the server executes, parses, stores, or fetches: SQL, command, HTTP stack, template, URL fetch, and other sinks."
keywords:
  - injection
  - SQL injection
  - command injection
  - SSRF
---

# Injection

**Injection** is organized by **sink** (what interprets tainted data): query languages, OS commands, inbound HTTP message handling, markup parsers, template engines, outbound URL fetches, mail assembly, file paths, logs, realtime channels, API dispatch, and expression runtimes. Classify by the dangerous **primitive**, not only by whether input arrived in the query string, header, or body.

## Child topics

| Area | Path |
|------|------|
| Database | [Database](database/index.md), SQL, NoSQL, ORM, LDAP |
| HTTP (inbound) | [HTTP](http/index.md), Smuggling / desync when front and back disagree |
| Outbound fetch | [Request forgery](request-forgery/index.md), SSRF and URL-layer abuse |
| Command | [Command](command/index.md) |
| Template engine | [Template engine](template-engine/index.md) |
| Markup | [Markups](markup/index.md) |
| Mail | [Mail](mail/index.md) |
| File | [File](file/index.md) |
| Log | [Log](log/index.md) |
| Realtime | [Realtime](realtime/index.md) |
| API dispatch | [API](api/index.md) |
| Expressions | [Expression evaluation](expression-evaluation/index.md) |
