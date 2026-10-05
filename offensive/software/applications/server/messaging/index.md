---
title: "Messaging"
description: "Offensive scope for enterprise messaging and mail servers: systems that authenticate users, sit on the perimeter, and hold both credentials and the trust to send as real people, with Microsoft Exchange as the primary on-premises target."
keywords:
  - messaging
  - mail server
  - Exchange
  - email
  - collaboration
---

# Messaging

Messaging and mail servers are high-value targets: they authenticate every user, they are usually reachable from the internet, they are often privileged in the surrounding directory, and the mailboxes they hold are full of credentials and sensitive intel. Compromising one frequently yields both a domain foothold and the ability to send as trusted employees.

## Products

- **[Microsoft Exchange](exchange/index.md)**: the dominant on-premises mail server, internet-facing and privileged in Active Directory, with a deep attack surface from enumeration through pre-auth RCE to Domain Admin.

## References

- [MITRE ATT&CK: email collection](https://attack.mitre.org/techniques/T1114/)
- [Black Hills InfoSec: attacking Exchange with MailSniper](https://www.blackhillsinfosec.com/attacking-exchange-with-mailsniper/)
