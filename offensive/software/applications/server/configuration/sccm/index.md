---
title: "SCCM / MECM: attacking Configuration Manager"
description: "The offensive surface of Microsoft Configuration Manager (SCCM/MECM): finding site systems in AD, harvesting the credentials it stores, taking over the site through NTLM relay, and abusing application and script deployment to run code as SYSTEM on every managed device."
keywords:
  - SCCM
  - MECM
  - Configuration Manager
  - site takeover
  - SharpSCCM
---

# SCCM / MECM

Microsoft Configuration Manager (SCCM, rebranded MECM) manages software, updates, and settings for most large Windows estates. An agent runs as **SYSTEM on every managed client**, a central **site server** orchestrates it, and the whole thing is registered in Active Directory. That makes it, in the words of the researchers who mapped it, an **enterprise C2 framework** already installed in the environment: take control of it and you can run code as SYSTEM everywhere, or pull the credentials it holds.

## How the attack surface grew

The offensive understanding of SCCM evolved in stages, and each stage is still useful depending on how the environment is configured:

- **2022, credentials first**: the original "easy win" was extracting the **Network Access Account** from client policy, a domain credential handed to any machine that asked. PXE boot media gave another unauthenticated path to the same secrets.
- **Microsoft's response**: Enhanced HTTP and the push away from the NAA reduced that path, so attention moved to the control plane.
- **2023 onward, takeover**: NTLM **relay** against the site server and SMS Provider, landing on the site database, grants the **Full Administrator** role, which is SYSTEM-level execution on every client. This is the current center of gravity, catalogued by SpecterOps' Misconfiguration Manager (RECON, CRED, TAKEOVER, EXEC, ELEVATE).

## Pages

- **[Reconnaissance](reconnaissance.md)**: locating site servers, management points, and distribution points from AD.
- **[Credential harvesting](credential-harvesting.md)**: Network Access Account, PXE media, and policy secrets.
- **[Site takeover](site-takeover.md)**: NTLM relay to the site database or SMS Provider for Full Administrator.
- **[Application deployment](application-deployment.md)**: running code as SYSTEM across device collections.

## References

- [SpecterOps: Misconfiguration Manager](https://github.com/subat0mik/Misconfiguration-Manager)
- [SharpSCCM (Mayyhem)](https://github.com/Mayyhem/SharpSCCM)
- [DEF CON 32: offensive SCCM, abusing Microsoft's C2 framework](https://infocondb.org/con/def-con/def-con-32/offensive-sccm-abusing-microsofts-c2-framework)
