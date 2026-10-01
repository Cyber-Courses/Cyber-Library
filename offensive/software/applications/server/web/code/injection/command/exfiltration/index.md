---
title: "Blind command injection exfiltration"
description: "Recovering output when none is returned: time-based oracles and out-of-band DNS/HTTP channels."
keywords:
  - blind command injection
  - out-of-band
  - OOB exfiltration
  - time-based
  - DNS exfiltration
---

# Exfiltration

When a command runs but its output never appears in the response, data is recovered through side channels. A **time-based** oracle makes execution conditional on a guess and measures a deliberate delay; an **out-of-band** channel makes the target reach a host you control (DNS or HTTP) and carries command output in the hostname or request path. These are the same inference techniques used in blind SQL injection, applied to shell output.
