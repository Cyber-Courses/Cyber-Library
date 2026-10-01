---
title: "Hardware: offensive techniques across the physical computing stack"
description: "The hardware category organizes attacks by the layer of the physical computing stack they target, from silicon and board up through firmware, platform, and architecture."
keywords:
  - hardware hacking
  - firmware attacks
  - silicon security
  - board level attacks
  - hardware architecture
---

# Hardware

The hardware category covers offensive work against physical computing devices and the low-level code that runs closest to them. These attacks assume physical or near-physical access to a device and exploit the fact that trust boundaries weaken the lower you go: a secret protected by software can often be read straight off a bus, a flash chip, or a debug port.

## Why it is split this way

The subcategories follow the layers of the hardware stack, because the access, tooling, and primitives change at each layer:

- **Silicon**: the chip itself, including side channels, fault injection, and attacks on on-die secrets and secure elements.
- **Board**: the circuit board and its interconnects, where bus sniffing, debug interfaces (JTAG, SWD), and chip-off recovery live.
- **Firmware**: the code stored on the device (BIOS/UEFI, embedded firmware, microcontroller images), extracted, analyzed, and modified.
- **Platforms**: integrated device classes and their specific ecosystems, where the attack combines several layers against a real product.
- **Architecture**: the design-level properties (instruction sets, memory models, trusted execution) that shape what the lower layers expose.

Organizing by layer matches how an assessment actually proceeds, from the outside in: identify the platform, pull the firmware, probe the board, and reach the silicon only when the layers above do not yield. Each layer needs different equipment and a different mental model, so keeping them separate stops a logic-analyzer technique from being filed next to a power-analysis one. This category is distinct from [physical](../physical/index.md) security, which is about entering spaces and defeating barriers rather than attacking devices.

## References

- [OWASP Firmware Security Testing Methodology](https://github.com/scriptingxss/owasp-fstm)
- [NIST: Hardware Security](https://csrc.nist.gov/)
