---
title: "File upload abuse"
description: "Turning an upload feature into code execution or file overwrite by defeating the checks on name, type, content, and destination of attacker-supplied files."
keywords:
  - file upload
  - web shell
  - unrestricted upload
  - zip slip
  - arbitrary file write
---

# Upload

Upload abuse starts from any feature that writes attacker-supplied bytes to disk: avatars, document attachments, import wizards, profile backgrounds.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess.

The prize is control over three properties of the stored file: its **name and extension** (so the server executes it), its **declared and real content** (so it passes type checks yet still runs), and its **destination path** (so it lands where it is served or where it overwrites something important). Unrestricted upload chains those first two to drop a web shell into a web-served directory and then requests it. Archive extraction (Zip Slip) attacks the third: entry names inside a crafted archive escape the unpack directory to write wherever the process can, overwriting assets, binaries, or configuration. Both routes commonly end in remote code execution.
