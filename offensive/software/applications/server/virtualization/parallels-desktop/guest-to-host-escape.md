---
title: "Guest to host escape: breaking out of Parallels Desktop"
description: "Escaping a Parallels Desktop guest to the macOS host through its emulated devices and integration services: the graphics and ToolGate interface, network and USB controllers, which run in host-side processes and parse guest-controlled input."
keywords:
  - Parallels escape
  - ToolGate
  - device emulation
  - macOS
  - guest to host
---

# Guest to host escape

Parallels runs each VM through host-side processes that emulate the guest's devices and expose integration interfaces (notably the ToolGate channel used by Parallels Tools). These parse guest-controlled input on the host, so memory-corruption flaws in the graphics, network, USB, or ToolGate handlers let a guest execute code on the macOS host.

```text
Parallels guest escape surfaces (reachable from a guest):
- The ToolGate interface (Parallels Tools integration channel)
- Graphics / display adapter
- Network and USB controllers
```

## Exploitation notes

- The ToolGate interface and the graphics path are recurring, productive targets, especially when Parallels Tools is installed in the guest.
- Escapes land on the macOS host as the user running Parallels, then escalate with a separate macOS local privilege escalation.
- Named instances are under [Known escape exploits](known-escape-exploits.md).

## References

- [Zero Day Initiative: Parallels research](https://www.zerodayinitiative.com/blog)
- [Parallels Desktop release notes](https://www.parallels.com/products/desktop/resources/)
