---
title: "Mailbox access: reading and impersonating mailboxes"
description: "Post-compromise use of Exchange mailboxes: searching your own and other users' mail for credentials and intel, granting ApplicationImpersonation to read every mailbox, and sending as trusted users for internal phishing."
keywords:
  - mailbox
  - ApplicationImpersonation
  - EWS
  - MailSniper
  - internal phishing
---

# Mailbox access

Mailboxes are a target in their own right: they hold passwords, password-reset links, VPN and app secrets, network diagrams, and the trust to send mail **as** a real employee. With one credential you read your own mail; with the right Exchange role you read **everyone's**.

## Searching mail

```powershell
# MailSniper: search your own mailbox for secrets
Invoke-SelfSearch -Mailbox user@example.local -Terms "password","vpn","secret"

# With the ApplicationImpersonation role, search every mailbox in the org
Invoke-GlobalMailSearch -ImpersonationAccount user -ExchHostname <exch> -Terms "password"
```

## Reading everyone: ApplicationImpersonation

The **`ApplicationImpersonation`** RBAC role lets an account act as any mailbox over EWS. If you reach an Exchange admin (or compromise the server), grant it to a controlled account and read the whole organisation's mail:

```powershell
# As an Exchange admin: grant impersonation, then search globally
New-ManagementRoleAssignment -Role ApplicationImpersonation -User attacker
```

## Sending as a user

Beyond reading, mailbox access enables **internal phishing** from a trusted sender: replying within real threads, or sending from an executive's address, defeats the usual "external sender" suspicion. Transport and inbox **rules** can also be planted for persistence (auto-forward, or triggering on a keyword).

## Exploitation notes

- Mail search is often the **fastest path to more credentials**: helpdesk resets, service-account passwords, and app secrets are routinely emailed in cleartext.
- `ApplicationImpersonation` is quieter than dumping the mailbox store and does not need SYSTEM on the server, only the role.
- Auto-forward or client rules are durable **persistence** and data exfiltration that survive a password reset of the victim.
- Sending as a trusted internal user is a strong pivot for lateral social-engineering once you hold one mailbox.

## Tools

- **MailSniper** (`Invoke-SelfSearch`, `Invoke-GlobalMailSearch`, `Add-MailboxPermission`): search and impersonation.
- **ruler**: inbox-rule and form-based mailbox persistence.
- **Exchange Management Shell / EWS**: role grants and mailbox operations.

## References

- [MailSniper (dafthack)](https://github.com/dafthack/MailSniper)
- [Black Hills InfoSec: attacking Exchange with MailSniper](https://www.blackhillsinfosec.com/attacking-exchange-with-mailsniper/)
- [Microsoft: ApplicationImpersonation role](https://learn.microsoft.com/en-us/exchange/client-developer/exchange-web-services/impersonation-and-ews-in-exchange)
