---
title: "Writable share poisoning: abusing a writable SMB share"
description: "Abusing a writable SMB share to act on other users: planting SCF, LNK, and URL files whose icons coerce authentication, poisoning Office documents and templates that run code when opened, and planting executables and DLLs that users or services run."
keywords:
  - writable share
  - SCF
  - LNK
  - DLL planting
  - share poisoning
---

# Writable share poisoning

A writable share is a staging ground for attacks on everyone who uses it. Three families matter: icon-loading files (SCF, LNK, URL) that coerce a browsing user's machine to authenticate to the attacker, Office documents and templates that run code when a user opens them, and executables and DLLs that users or services on the share run directly.

## Subtopics

- **[SCF and LNK coercion](scf-and-lnk-coercion.md)**: coerce authentication on browse.
- **[Office and template poisoning](office-and-template-poisoning.md)**: run code on open.
- **[Executable and DLL planting](executable-and-dll-planting.md)**: replace or sideload binaries.

## References

- [HackTricks: pentesting SMB](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html)
- [The Hacker Recipes: forced authentication](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications)
