---
title: "Kerberos: roasting, forging, and reusing tickets"
description: "Abusing Kerberos in Active Directory: roasting service and pre-auth-less accounts for crackable material, reusing keys and tickets, forging tickets outright, abusing delegation, and the shadow-credentials and sAMAccountName-spoofing paths to impersonation."
keywords:
  - kerberos
  - kerberoasting
  - golden ticket
  - delegation
  - pass the ticket
---

# Kerberos

Kerberos is the primary authentication protocol in Active Directory, and almost every high-value AD attack ends up in Kerberos terms. Its design gives an attacker several distinct openings: the encrypted parts of tickets are derived from an account's key, so they can be **roasted** and cracked offline; the key itself authenticates, so it can be **reused** without the password; and whoever holds the `krbtgt` key or a service key can **forge** tickets at will.

## How the openings arise

A quick model of the protocol explains each attack:

- The client proves its identity to the KDC (AS exchange) and receives a **TGT** encrypted with the `krbtgt` key.
- It presents the TGT to get **service tickets** (TGS exchange), each encrypted with the target service account's key.
- The service trusts the ticket because it can decrypt it with its own key, and trusts the privileges inside (the PAC) because the KDC signed them.

Every trust in that chain is a target: the account key (roasting, pass-the-key), the TGT (pass-the-ticket, golden ticket), the service key (silver ticket), the PAC (diamond/sapphire tickets), and the delegation flags that let one service request tickets *as* another user.

## Pages

- **[SPN discovery](spn-discovery.md)**: finding the service accounts that Kerberoasting targets.
- **[Roasting](roasting.md)**: Kerberoasting, AS-REP roasting, and timeroasting, obtaining crackable material from the protocol.
- **[Pass-the-key and overpass-the-hash](pass-the-key-and-overpass-the-hash.md)**: turning a key or hash into a TGT.
- **[Pass-the-ticket](pass-the-ticket.md)**: extracting and injecting tickets to reuse them.
- **[Forged tickets](forged-tickets.md)**: golden, silver, diamond, and sapphire ticket forgery.
- **[MS14-068](ms14-068.md)**: forging a privileged PAC, any user to Domain Admin on an unpatched DC.
- **[Delegation](delegation/index.md)**: unconstrained, constrained, and resource-based constrained delegation abuse.
- **[Kerberos relay](kerberos-relay.md)**: capturing TGTs via unconstrained delegation and relaying Kerberos to LDAP or AD CS.
- **[Shadow credentials](shadow-credentials.md)**: adding key credentials to impersonate via PKINIT.
- **[UnPAC-the-hash](unpac-the-hash.md)**: recovering the NT hash from a PKINIT authentication.
- **[sAMAccountName spoofing](samaccountname-spoofing.md)**: the noPac privilege-escalation path.

## References

- [GhostPack Rubeus: raw Kerberos abuse toolkit](https://github.com/GhostPack/Rubeus)
- [Impacket: Kerberos example scripts](https://github.com/fortra/impacket)
- [Microsoft: Kerberos authentication](https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview)
