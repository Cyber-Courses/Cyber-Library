---
title: "Writable share poisoning: planting payloads and credential-coercion files"
description: "A writable SMB share is an execution and credential-capture primitive. An attacker plants executables or DLLs where they will be run, poisons Office documents and templates users open, and drops SCF and LNK files whose icons force the viewer to authenticate to an attacker host. Write access turns a file server into code execution and captured credentials."
keywords:
  - writable share
  - dll planting
  - scf lnk
  - template poisoning
  - credential coercion
---

# Writable share poisoning

Write access to a share is far more than the ability to store files: it is a way to get code executed and credentials captured when other users or systems interact with the share. Three techniques follow. Planting executables or DLLs where an application or user will run them gives code execution in that context. Poisoning Office documents and their templates runs macros or external references when a user opens them. And dropping SCF or LNK files whose icon or target points at an attacker host forces the viewer's machine to authenticate there as soon as the folder is browsed, capturing or relaying the credential. The common thread is that write access converts normal use of the share into attacker advantage.

```bash
# confirm writable, then plant (null/guest/creds per the enumeration)
smbclient //<target>/<share> -U 'user%pass' -c 'put payload'
nxc smb <target> -u user -p 'pass' --shares     # WRITE flag confirms the target
```

## Subtopics

- **[Executable and DLL planting](executable-and-dll-planting.md)**: getting planted code run.
- **[Office and template poisoning](office-and-template-poisoning.md)**: macro and template execution on open.
- **[SCF and LNK coercion](scf-and-lnk-coercion.md)**: forcing authentication on folder browse.

## References

- [MITRE ATT&CK: taint shared content](https://attack.mitre.org/techniques/T1080/)
- [NetExec: SMB](https://www.netexec.wiki/smb-protocol)
