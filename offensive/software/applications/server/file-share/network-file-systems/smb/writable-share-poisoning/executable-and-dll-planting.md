---
title: "Executable and DLL planting: hijacking programs run from a share"
description: "Abusing a writable SMB share that users or services execute from, by replacing or adding executables, or by planting a DLL that an executable on the share loads through search-order or sideloading, so the attacker's code runs in the victim's context."
keywords:
  - DLL sideloading
  - DLL search order
  - binary planting
  - writable share
  - execution
---

# Executable and DLL planting

Some writable shares host programs that users launch or that services run on a schedule. Replacing or adding an executable there runs attacker code when it is next used. More subtly, planting a DLL next to an executable on the share, one it loads by search order or sideloading, runs code in that program's context without modifying the executable itself.

```bash
# Replace or add a binary users run from the share
cp payload.exe '//share/Tools/update.exe'
# Or sideload: drop a malicious DLL the share's EXE loads from its own directory
cp evil.dll '//share/App/version.dll'       # loaded by App.exe via search order
```

## Exploitation notes

- Target programs that run with higher privilege than the planter: a service that executes from the share, or an admin's tool.
- DLL sideloading is stealthier than replacing the EXE, since the signed executable is untouched; identify which DLLs the share's binaries resolve relatively.
- Scheduled tasks and logon scripts that point at a writable share path are reliable execution triggers.

## References

- [HackTricks: DLL hijacking](https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/dll-hijacking/index.html)
- [Microsoft: dynamic-link library search order](https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order)
