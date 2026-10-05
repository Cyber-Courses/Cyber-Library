---
title: "RealVNC auth bypass: forcing the None security type"
description: "A RealVNC flaw let a client dictate the security type during the RFB handshake, so a malicious client selected None even though the server required a password. The server honoured the client's choice and granted an unauthenticated session, a complete authentication bypass against affected RealVNC versions."
keywords:
  - realvnc
  - auth bypass
  - security type
  - none
  - handshake
---

# RealVNC auth bypass

RealVNC shipped a flaw in the RFB security-type negotiation: the protocol intends the server to offer a set of security types and the client to pick one of the offered ones, but affected RealVNC versions let the client select a type the server had not offered, specifically "None" (type 1). A malicious client, after receiving the server's list, simply announced that it would use None, and the server proceeded to grant an unauthenticated session, skipping the password entirely. This is a complete pre-auth bypass against the affected versions, giving an interactive desktop without any credential.

```bash
# detect affected RealVNC servers
nmap -p5900 --script realvnc-auth-bypass <target>     # reports vulnerable + grants a session
# the exploit: complete the RFB handshake but respond with security type 1 (None)
# even when the server offered only type 2 (VNC auth); affected servers accept it.
```

## Exploitation notes

- The bug is the server trusting the client's chosen security type without checking it was offered; the attack is to force None where the server required a password, so the fix was to validate the client's selection.
- `realvnc-auth-bypass` both detects and demonstrates it by obtaining a session; a positive result is immediate unauthenticated desktop access.
- This is version-specific to the affected RealVNC builds; fingerprint the [implementation](../enumeration/implementation-detection.md) and version first.
- It is the cleanest VNC access when present, no brute force or capture needed; where not present, fall back to the password and no-auth routes.

## References

- [RealVNC security-type bypass advisory](https://www.realvnc.com/en/connect/docs/)
- [nmap realvnc-auth-bypass](https://nmap.org/nsedoc/scripts/realvnc-auth-bypass.html)
