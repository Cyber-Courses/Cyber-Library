---
title: "macOS: attacking the macOS host platform"
description: "Offensive techniques against a macOS host: local privilege escalation, bypassing the TCC privacy framework, Gatekeeper and code-signing, and the System Integrity Protection and app sandbox boundaries, reading the keychain, and persistence through launch agents and daemons."
keywords:
  - macOS privilege escalation
  - TCC bypass
  - Gatekeeper
  - SIP
  - keychain
---

# macOS

macOS layers several of its own controls on top of a Unix base, so attacking a Mac is as much about defeating Apple's privacy, signing, and integrity frameworks as it is about classic Unix privilege escalation. The work is to move from a normal user to `root`, step past the controls that gate sensitive data and code execution, and read the secrets the system protects.

## The local surface

- **Privilege escalation**: the Unix mechanisms (SUID, `sudo`, writable paths) plus macOS-specific service, helper-tool, and installer abuses.
- **TCC**: bypassing Transparency, Consent, and Control, the framework that gates access to files, the camera, the microphone, and automation, to reach data without the user's approval.
- **Gatekeeper and code signing**: defeating quarantine, notarization, and signature checks to run unsigned or untrusted code.
- **SIP and the sandbox**: escaping the application sandbox and the boundaries System Integrity Protection enforces even on `root`.
- **Credential access**: reading the login and system **keychains** and other credential stores.
- **Persistence**: launch agents and daemons, login items, and configuration profiles.

## References

- [Apple Platform Security guide](https://support.apple.com/guide/security/welcome/web)
- [MITRE ATT&CK: macOS](https://attack.mitre.org/matrices/enterprise/macos/)
