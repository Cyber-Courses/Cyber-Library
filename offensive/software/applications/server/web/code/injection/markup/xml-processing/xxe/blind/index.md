---
title: "Blind XXE without reflected output"
description: "When the parser resolves entities but returns nothing parsed, data is recovered through error messages, timing, and out-of-band channels."
keywords:
  - blind XXE
  - no reflected output
  - error-based XXE
  - out-of-band channel
  - parameter entity
---

# Blind

Blind XXE covers the common case where the application parses attacker XML with external entities enabled but never places any parsed value in the response. The in-band read of `&xxe;` produces nothing, because the field it would land in is not echoed, discarded after validation, or consumed by a backend that returns only a status.

The vulnerability is still fully exploitable; the data just has to leave through a side channel. Three channels do the work. **Out-of-band** retrieval makes the parser send file contents to an attacker-controlled server over HTTP or FTP. **Error messages** provoke the parser into including file contents inside a fatal-error string that the application happens to surface. **Timing** distinguishes outcomes when even errors are suppressed. All of them rely on **parameter entities** and, in the portable form, an external DTD, because general-entity nesting is rejected inside the internal subset by most parsers.

This subtree covers the error-based channel and the local-DTD technique for environments where outbound network access is blocked.

## Pages

- **[Error messages](error-messages.md)**: A parameter entity that feeds file contents into an invalid SYSTEM path forces a parse error whose message leaks the file.
- **[Local DTD](local-dtd.md)**: Repurposing a DTD file already on disk lets a blind XXE redefine an internal entity and trigger an error-based leak when outbound network is blocked.

## Tools

- **XXEinjector**: automating blind XXE through out-of-band and error-based channels.
- **Burp Collaborator**: capturing out-of-band callbacks that confirm blind XXE.
- **Burp Suite**: crafting parameter-entity payloads in Repeater.

## References

- PortSwigger Web Security Academy: Blind XXE injection
- OWASP: XML External Entity Prevention Cheat Sheet
