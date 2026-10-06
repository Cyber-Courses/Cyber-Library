---
title: "Configuration"
order: 11
description: "Offensive scope for enterprise configuration-management platforms: systems that push software, scripts, and settings to managed endpoints at scale, and therefore hand an attacker code execution across the estate once abused."
keywords:
  - configuration management
  - SCCM
  - MECM
  - endpoint management
  - software deployment
---

# Configuration

Configuration-management platforms exist to push software, scripts, and settings to thousands of managed machines from one console. That is also precisely what an attacker wants: **code execution across the whole estate** from a single compromised component. These systems are deeply trusted, run agents as SYSTEM on every client, hold or broker credentials, and are usually integrated with Active Directory, so abusing one is often equivalent to owning the domain's endpoints.

This area covers the offensive surface of those platforms: how to find them, pull the credentials they store, take over their control plane, and turn their own deployment features into mass code execution.

## Products

- **[SCCM / MECM](sccm/index.md)**: Microsoft Configuration Manager, the dominant on-premises platform and the richest attack surface of the group.
- **[WSUS](wsus.md)**: Windows Server Update Services, abused to push a malicious update that runs as SYSTEM on managed clients.

## References

- [SpecterOps: Misconfiguration Manager, the SCCM attack and defense knowledge base](https://github.com/subat0mik/Misconfiguration-Manager)
- [Palo Alto Networks: SCCM, enterprise backbone or attack vector](https://www.paloaltonetworks.com/blog/security-operations/sccm-enterprise-backbone-or-attack-vector/)
