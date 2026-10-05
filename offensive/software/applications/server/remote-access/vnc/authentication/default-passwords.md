---
title: "Default passwords: vendor and guessable VNC passwords"
description: "VNC on appliances and preinstalled tools frequently uses a fixed or guessable password, and the classic VNC password is capped at eight characters, shrinking the space further. Trying vendor defaults and common VNC passwords often yields desktop access, and stored VNC password files can be decrypted with the fixed, published DES key."
keywords:
  - vnc default password
  - eight character limit
  - stored password
  - vncpasswd
  - fixed key
---

# Default passwords

Where a VNC server does use the password scheme, that password is often weak: appliances, KVM-over-IP devices, and bundled remote tools ship with fixed or documented defaults, and the classic VNC password is truncated to eight characters, shrinking the keyspace. Trying the device's default and common VNC passwords frequently works. Separately, VNC stores its password on disk obfuscated with a fixed, publicly known DES key (not a hash), so a recovered password file is decrypted directly to the plaintext password, which is then reused.

```bash
# try defaults/common passwords (8-char cap means short lists are effective)
vncviewer <target>::5900        # enter documented default or common password
nmap -p5900 --script vnc-brute --script-args passdb=vnc-defaults.txt <target>
# decrypt a stored VNC password file (fixed DES key, reversible)
#   files: ~/.vnc/passwd, UltraVNC.ini, registry values
echo -n "<hex-of-passwd-file>" | xxd -r -p | openssl enc -d -des-cbc -K e84ad660c4721ae0 -iv 0000000000000000 2>/dev/null
vncpwd passwd      # tools that apply the known key directly
```

## Exploitation notes

- The eight-character truncation means only the first eight bytes of any password matter, so short default/common-password lists are disproportionately effective; identify the device for its documented default.
- Stored VNC passwords are obfuscated, not hashed: the DES key is fixed and public, so any recovered `passwd` file, `UltraVNC.ini`, or registry value decrypts straight to the plaintext with a known-key tool.
- A decrypted VNC password is reusable, test it on other VNC endpoints and consider it for other services, since operators reuse it.
- Where brute force is needed rather than defaults, see [Password brute force](password-brute-force.md); where the server has no password, see [No authentication](no-authentication.md).

## Tools

- [vncpwd (fixed-key VNC password decryptor)](https://github.com/jeroennijhof/vncpwd)

## References

- [VNC password storage and fixed key](https://www.vidarholen.net/contents/junk/vnc.html)
- [HackTricks: VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
