---
title: "WSUS: pushing a malicious update to SYSTEM"
description: "Abusing Windows Server Update Services to run code as SYSTEM on managed clients: approving a malicious update from a compromised WSUS server, and injecting one in-path when WSUS is served over plain HTTP."
keywords:
  - WSUS
  - SharpWSUS
  - WSUSpendu
  - malicious update
  - SYSTEM
---

# WSUS

Windows Server Update Services distributes updates that clients install as **SYSTEM**, which is exactly the primitive an attacker wants: approve the right "update" and you run code as SYSTEM on every targeted machine. There are two routes, one from a compromised WSUS server and one from the network when WSUS is served over plain HTTP.

## Approving a malicious update

With admin on the WSUS server, create an update that runs a **Microsoft-signed binary** with attacker-chosen arguments, and approve it for a target computer group. The signing constraint is met by living-off-the-land binaries already trusted (for example `PsExec.exe`), driven with command-line arguments to add an admin or run a payload:

```text
# SharpWSUS: inspect, create, approve, then clean up
SharpWSUS.exe inspect
SharpWSUS.exe create /payload:"C:\PsExec64.exe" /args:"-accepteula -s cmd /c \"net user ...\"" /title:"Update"
SharpWSUS.exe approve /updateid:<guid> /computername:<target> /groupname:"Group"
```

`WSUSpendu` is the original PowerShell approach, injecting the approval directly into the WSUS database, useful where the console workflow is blocked.

## In-path injection over HTTP

Where WSUS is configured over **HTTP** rather than HTTPS, a man-in-the-middle can inject a malicious update to clients, since the metadata is not authenticated at the transport. Positioned between client and server, deliver a signed-binary "update" to every client that checks in:

```text
# PyWSUS: serve a malicious update to clients whose WSUS URL is http:// (works on modern clients)
# WSUSpect is legacy (tested on Windows 7/8, not Windows 10/11), so prefer PyWSUS for modern targets
# combine with an MITM primitive (ARP, rogue DHCP/WPAD, mitm6) to reach the client's update traffic
```

## Exploitation notes

- The payload must be **Microsoft-signed**, so these attacks pair a trusted LOLBIN (PsExec, BgInfo) with its command-line arguments rather than dropping an unsigned executable.
- A WSUS server often updates **many** clients and sometimes other servers and DCs, so a single approval is estate-wide SYSTEM, much like SCCM [application deployment](sccm/application-deployment.md).
- The **HTTP** variant needs no WSUS compromise, only a network MITM to the client's update traffic, so test whether WSUS runs over HTTP during recon.
- Clean up the approval and update afterwards (SharpWSUS `delete`), since an orphaned update is both noisy and disruptive to the environment.

## Tools

- **SharpWSUS** (LRQA/Nettitude): inspect, create, approve, and delete malicious updates from a compromised server.
- **WSUSpendu**: PowerShell injection of an approval into the WSUS database.
- **PyWSUS**: in-path injection against HTTP WSUS clients, including modern Windows.
- **WSUSpect**: the original in-path proxy, legacy only (Windows 7/8).

## References

- [LRQA/Nettitude: introducing SharpWSUS](https://www.lrqa.com/en/cyber-labs/introducing-sharpwsus/)
- [GoSecure: PyWSUS, WSUS man-in-the-middle](https://github.com/GoSecure/pywsus)
- [h4ms1k: red teaming WSUS](https://h4ms1k.github.io/Red_Team_WSUS/)
