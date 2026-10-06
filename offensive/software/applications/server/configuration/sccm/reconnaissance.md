---
title: "SCCM reconnaissance: finding the site systems"
description: "Discovering a Configuration Manager deployment from Active Directory and the network: the System Management container, management points, distribution points, and the site server, using sccmhunter and SharpSCCM before any privileged access."
keywords:
  - SCCM reconnaissance
  - System Management container
  - management point
  - sccmhunter
  - SharpSCCM
---

# Reconnaissance

Before anything else you need to know whether SCCM is present and which machines play which role: the **site server** (the prize), the **management points** (where clients fetch policy), the **distribution points** (content, including PXE), and the **site database** (MSSQL). Most of this is discoverable with a single domain credential, because SCCM publishes itself into Active Directory.

## From Active Directory

When the schema is extended, SCCM publishes to the **System Management** container, and roles are findable by naming and SPNs:

```bash
# sccmhunter: pull SCCM assets from the System Management container and fuzzy-match MECM/SCCM hosts
sccmhunter.py find -u user -p password -d example.local -dc-ip <dc>
sccmhunter.py show -all          # dump what was found (site servers, MPs, DPs)
```

```text
# Manually: the container and the MSSQL/management SPNs reveal the roles
LDAP: CN=System Management,CN=System,DC=example,DC=local
SPNs: MSSQLSvc/... on the site database, HTTP/... on management points
```

## From the network

```bash
# SharpSCCM: identify the management point and site code from a client's perspective
SharpSCCM.exe local site-info
SharpSCCM.exe get site-info -d example.local   # queries a DC over LDAP
```

## Exploitation notes

- The **site server** is the target for [takeover](site-takeover.md); the **management points** are where [policy credentials](credential-harvesting.md) are fetched; the **distribution points** may offer PXE.
- Schema extension means recon often needs **only a low-privilege domain user**, so this maps the whole hierarchy before you hold anything privileged.
- Note the **site code** and whether a central administration site sits above the primary, since takeover scope follows the hierarchy.

## Tools

- **sccmhunter** (garrettfoster13): AD-based discovery of SCCM assets and profiling.
- **SharpSCCM** (Mayyhem): client-side and management-point enumeration.
- **ldapsearch / BloodHound**: the System Management container and SPNs directly.

## References

- [SpecterOps: Misconfiguration Manager, RECON](https://github.com/subat0mik/Misconfiguration-Manager/tree/main/attack-techniques/RECON)
- [sccmhunter (garrettfoster13)](https://github.com/garrettfoster13/sccmhunter)
- [Truesec: common SCCM misconfigurations leading to privilege escalation](https://www.truesec.com/hub/blog/sccm-tier-killer)
