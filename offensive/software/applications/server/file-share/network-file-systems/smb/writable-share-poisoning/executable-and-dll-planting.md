---
title: "Executable and DLL planting: getting planted code run from a share"
description: "Writing executables or DLLs to a share leads to code execution when something runs them: replacing or adding programs in a deployment or application share, and DLL planting where an application loads a library by name from a share-relative path. The payload runs in the context of whatever executes it, often a service account or an administrator."
keywords:
  - dll planting
  - dll hijacking
  - deployment share
  - executable planting
  - search order
---

# Executable and DLL planting

Writing program files to a share only matters when something executes them, and shares provide several such triggers. Deployment, software-distribution, and application shares hold executables that machines or users run, so replacing or adding one runs attacker code on those machines. DLL planting (hijacking) exploits how applications locate libraries: when a program loads a DLL by name and searches directories including its own folder on a share, placing a malicious DLL of that name there gets it loaded into the process. The payload runs as whoever executes it, frequently a service account or an administrator deploying software.

## Routes

```bash
# 1. deployment / application share: add or replace an executable that gets run
smbclient //<t>/deploy -U 'user%pass' -c 'put evil.exe setup.exe'
# 2. DLL planting: identify a DLL an app loads from its (share-based) directory,
#    then plant a malicious DLL of that name next to the executable
#    - the app's import table / a procmon trace shows which DLLs it looks for
smbclient //<t>/apps -U 'user%pass' -c 'cd AppDir; put evil.dll version.dll'
```

For DLL planting, the target is a DLL the application loads by name and resolves through a search order that includes the application's own directory (on the share). Common hijackable names are those the app expects to find locally; a malicious DLL exporting the required functions (or a proxy that forwards to the real one) runs its `DllMain` in the process.

## Exploitation notes

- The payload inherits the executing identity: a deployment share executed by machines as SYSTEM, or an application launched by an administrator, makes planting a privilege win, not just execution.
- DLL planting needs the application to load a DLL by name with a search path that reaches the writable share location; a proxy/forwarder DLL preserves the app's function so it keeps working while your code runs.
- Prefer a file the environment actually runs on a schedule or at login/deploy, so execution is deterministic rather than opportunistic.
- Distinct from coercion (which captures credentials on mere browsing), this requires the file to be executed; pair with [SCF and LNK coercion](scf-and-lnk-coercion.md) when you only get a user to view the folder.

## References

- [MITRE ATT&CK: DLL search order hijacking](https://attack.mitre.org/techniques/T1574/001/)
- [MITRE ATT&CK: taint shared content](https://attack.mitre.org/techniques/T1080/)
