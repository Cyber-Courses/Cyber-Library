---
title: "Unattended access: abusing always-on remote-desktop access"
description: "Unattended access lets a remote-desktop tool be connected to at any time without a user present, authenticated by a fixed unattended password. When that password is weak, default, reused, or recovered from the client's stored configuration, an attacker connects to the machine whenever they like, gaining persistent interactive control with no one aware."
keywords:
  - unattended access
  - unattended password
  - persistent access
  - teamviewer
  - anydesk
---

# Unattended access

Unattended access is the feature that makes these tools a remote-administration backbone: it allows connecting to a machine at any time, with no user present to approve or read out a session password, authenticated instead by a fixed unattended-access password configured on the client. For an attacker this is standing, persistent control if they obtain that password, which they do by guessing it (often weak or reused), finding it set to a default, or recovering it from the client's stored configuration on a machine they already touched. Once held, the attacker connects whenever they choose and controls the machine interactively, and this is also a stealthy persistence mechanism.

```bash
# the unattended password is stored client-side; recover it where you have local access
#   TeamViewer/AnyDesk store configuration (and in some versions recoverable secrets)
#   in the registry / app data; dump and decrypt per the product's storage
reg query 'HKLM\SOFTWARE\WOW6432Node\TeamViewer' 2>/dev/null       # (example location)
# with the unattended ID + password, connect at will from the attacker client
```

## Exploitation notes

- Unattended access turns the tool into a durable backdoor: a recovered or weak unattended password grants connection at any time, so it is both access and persistence, quieter than planting new tooling.
- The unattended password is stored on the client; where you already have local access (even briefly), recovering it from the product's configuration/registry gives durable remote re-entry, sometimes recoverable in plaintext-equivalent form depending on the version.
- Weak or default unattended passwords are guessable directly; reuse means a password found elsewhere may be the unattended one.
- This is the highest-value authentication outcome because it is standing access; combine with [weak passwords](weak-passwords.md) and [ID-based access](id-based-access.md) to reach it.

## References

- [TeamViewer unattended access](https://www.teamviewer.com/en/)
- [AnyDesk unattended access](https://anydesk.com/en/)
