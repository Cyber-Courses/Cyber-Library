---
title: "RealVNC authentication bypass: CVE-2006-2369"
description: "CVE-2006-2369 affected RealVNC 4.1.0 and 4.1.1 and products embedding the vulnerable code. The server accepted a client-selected None security type even when it had not offered that type, allowing authentication bypass."
keywords:
  - realvnc
  - CVE-2006-2369
  - auth bypass
  - security type
  - none
  - handshake
---

# RealVNC auth bypass

CVE-2006-2369 affected RealVNC 4.1.0 and 4.1.1, as well as some products that embedded the vulnerable RealVNC code. During RFB security negotiation, the affected server failed to verify that the client's selected security type was one the server had offered. Selecting `None` (type 1) could therefore bypass the configured authentication check. Confirm the exact product and version before applying this historical finding.

```bash
# check for CVE-2006-2369 behavior; this reports the result but is not a VNC viewer
nmap -sV -p5900 --script realvnc-auth-bypass <target>
```

## Exploitation notes

- The flaw is the server accepting a security type it did not advertise.
- Nmap's script checks for the vulnerable negotiation behavior; it does not provide an interactive desktop session.
- A positive check indicates that a separate compatible VNC client could exploit the authentication bypass, subject to reachability and the embedded product's behavior.
- Treat other RealVNC releases and products as unaffected unless their own advisory says otherwise.

## References

- [nmap realvnc-auth-bypass](https://nmap.org/nsedoc/scripts/realvnc-auth-bypass.html)
- [CVE-2006-2369](https://www.cve.org/CVERecord?id=CVE-2006-2369)
