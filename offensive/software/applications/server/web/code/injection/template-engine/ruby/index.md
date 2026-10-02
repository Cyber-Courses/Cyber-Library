---
title: "Ruby server-side template injection"
description: "SSTI in Ruby template engines: ERB (and the ERB-compatible Erubi/Erubis) evaluating embedded Ruby directly for immediate command execution."
keywords:
  - Ruby SSTI
  - ERB
  - Erubi
  - Rails template injection
---

# Ruby

Ruby SSTI centers on ERB, the standard embedded-Ruby engine used by Rails and many standalone apps (with Erubi and Erubis as compatible variants). ERB evaluates Ruby inside its tags with no sandbox, so a template injection is direct code execution: there is no object graph to walk and no built-in to escape.

Detection uses ERB's own tag syntax rather than `{{7*7}}`, since ERB executes `<%= ... %>`. The escalation is immediate through Ruby's backticks, `system`, `IO.popen`, or `%x{}`.

## Engines

- **[ERB](erb.md)**: `<%= `command` %>` and the `system`/`IO.popen` variants, plus the Rails `render inline:` sink.

## References

- Ruby ERB and Erubi documentation
- PortSwigger Web Security Academy: Server-side template injection
