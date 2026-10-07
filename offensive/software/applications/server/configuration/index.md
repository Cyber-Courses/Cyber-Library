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

The Windows-native platforms:

- **[SCCM / MECM](sccm/index.md)**: Microsoft Configuration Manager, the dominant on-premises platform and the richest attack surface of the group.
- **[WSUS](wsus.md)**: Windows Server Update Services, abused to push a malicious update that runs as SYSTEM on managed clients.

The cross-platform configuration-management tools, each of which runs its agent or connection as root on every managed node:

- **[Ansible](ansible.md)**: agentless push over SSH; abuse the control node, Vault secrets, playbooks, and AWX/Tower.
- **[SaltStack](saltstack.md)**: master and minions over ZeroMQ; command the fleet from the master and exploit the unauthenticated request server.
- **[Puppet](puppet.md)**: pull model from the Puppet server; inject into the control repository and loot Hiera and PuppetDB.
- **[Chef](chef.md)**: pull model driven by knife credentials, cookbooks, data bags, and run lists.

## Scope

Microsoft Intune is a cloud-hosted endpoint manager, the cloud counterpart of SCCM. It is attacked as a vendor-hosted service and belongs with the cloud identity and endpoint-management surface under Online, not here.

## References

- [SpecterOps: Misconfiguration Manager, the SCCM attack and defense knowledge base](https://github.com/subat0mik/Misconfiguration-Manager)
- [Palo Alto Networks: SCCM, enterprise backbone or attack vector](https://www.paloaltonetworks.com/blog/security-operations/sccm-enterprise-backbone-or-attack-vector/)
