---
title: "Legacy VNC passwords: defaults and stored-password files"
description: "Assess documented device defaults and legacy VNC Authentication separately. Some vncpasswd-compatible implementations use only the first eight password bytes and store a reversibly obfuscated password file, but RFB does not define one universal on-disk format."
keywords:
  - vnc default password
  - eight character limit
  - stored password
  - vncpasswd
  - fixed key
---

# Default passwords

Default credentials are product and firmware specific; test only a documented default that applies to the identified device. Separately, implementations using the classic VNC Authentication scheme may use only the first eight password bytes. Legacy `vncpasswd`-compatible implementations such as TigerVNC also store an eight-byte reversibly obfuscated password file. That storage format belongs to the implementation, not the RFB protocol, and must not be assumed for RealVNC, UltraVNC, registry-backed products, or newer authentication types without verification.

```bash
# inspect a recovered legacy password file only after identifying its implementation
file "$HOME/.config/tigervnc/passwd"
# use an implementation-aware recovery tool rather than a generic OpenSSL recipe
vncpwd "$HOME/.config/tigervnc/passwd"
```

## Exploitation notes

- In classic VNC Authentication implementations, only the first eight password bytes may be significant; confirm the implementation before applying this limitation.
- TigerVNC documents its legacy password file as obfuscated rather than securely encrypted and warns that a reader can recover the plaintext.
- Do not apply the TigerVNC password-file method to `UltraVNC.ini`, registry values, or other products without format-specific evidence.
- Where brute force is needed rather than defaults, see [Password brute force](password-brute-force.md); where the server has no password, see [No authentication](no-authentication.md).

## Tools

- [vncpwd (fixed-key VNC password decryptor)](https://github.com/jeroennijhof/vncpwd)

## References

- [TigerVNC vncpasswd documentation](https://github.com/TigerVNC/tigervnc/blob/master/unix/vncpasswd/vncpasswd.man)
- [TigerVNC VNC authentication implementation](https://github.com/TigerVNC/tigervnc/blob/master/common/rfb/SSecurityVncAuth.cxx)
