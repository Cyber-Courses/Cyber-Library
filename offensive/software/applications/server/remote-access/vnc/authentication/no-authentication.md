---
title: "No authentication: VNC servers that require no password"
description: "A VNC server offering the None security type (type 1) grants an interactive desktop to anyone who connects, with no password. This is a common misconfiguration on helpdesk tools, KVM-over-IP devices, industrial systems, and VMs, and it is found simply by connecting or by reading the offered security types during enumeration."
keywords:
  - no authentication
  - security type none
  - open vnc
  - unauthenticated
  - desktop access
---

# No authentication

The simplest VNC weakness is a server that requires no authentication at all, offering RFB security type 1 ("None"). Anyone who connects gets the interactive desktop. This is widespread on systems set up for convenience: helpdesk and screen-sharing tools, KVM-over-IP and out-of-band console devices, industrial HMIs, kiosks, and quickly-provisioned VMs. It needs no attack, only reachability, and internet-wide scans continually catalogue open VNC desktops.

```bash
# the offered security type reveals it; type 1 = None
nmap -p5900-5905 --script vnc-info <target>    # look for "security types: None (1)"
# just connect; no password is requested
vncviewer <target>::5900
# search-engine discovery of open VNC (external): Shodan "port:5900 authentication disabled"
```

## Exploitation notes

- Enumeration is the detector: a security-type list containing `None (1)` means unauthenticated desktop access; often it is the only type offered.
- These are frequently high-value consoles, a KVM-over-IP card controls a server's hardware and BIOS, an HMI controls industrial equipment, a helpdesk tool is logged in as a user, so "no auth" can mean far more than one desktop.
- Scan the display range (5900-590x); multi-display and multi-VM hosts expose several VNC endpoints, not all configured alike.
- An open desktop is immediate interactive control; use it to read the screen, run commands in any open session, and pivot from the logged-in user's context.

## References

- [RFC 6143: None security type](https://datatracker.ietf.org/doc/html/rfc6143#section-7.2.1)
- [HackTricks: open VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
