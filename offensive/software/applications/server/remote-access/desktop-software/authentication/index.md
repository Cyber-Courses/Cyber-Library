---
title: "Authentication: attacking remote-desktop software access control"
description: "Third-party remote-desktop tools authenticate by a device ID plus a session or unattended-access password. The weaknesses are guessable or brute-forceable IDs and passwords, unattended-access configurations that allow passwordless or weak-password connection at any time, and reused or default passwords. Gaining access yields interactive control of the remote machine as its logged-in user."
keywords:
  - id-based access
  - unattended access
  - weak password
  - teamviewer
  - anydesk
---

# Authentication

These products authenticate a connection with a device identifier (the TeamViewer/AnyDesk ID) and a password, either a one-time session password shown to the present user or a configured unattended-access password for connecting without anyone at the keyboard. The attack surface follows. IDs are numeric and, combined with weak passwords, brute-forceable. Unattended access, set up for convenience, lets an attacker connect at any time, and where its password is weak, default, or absent, that is standing remote control. And passwords are reused and weak like any others. Success is full interactive control of the remote machine as its logged-in user, often with whatever privileges that user holds.

## Subtopics

- **[ID-based access](id-based-access.md)**: enumerating and targeting device IDs.
- **[Unattended access](unattended-access.md)**: abusing always-on passwordless or weak-password access.
- **[Weak passwords](weak-passwords.md)**: guessing session and unattended passwords.

## References

- [TeamViewer unattended access](https://www.teamviewer.com/en/)
- [HackTricks](https://book.hacktricks.xyz/)
