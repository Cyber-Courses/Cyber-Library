---
title: "Kerberos relay: forwarding Kerberos authentication"
description: "Relaying Kerberos authentication with krbrelayx: capturing TGTs through unconstrained delegation, and relaying coerced Kerberos service tickets to LDAP or AD CS when the attacker controls the service name the victim authenticates to."
keywords:
  - kerberos relay
  - krbrelayx
  - unconstrained delegation
  - AD CS
  - SPN
---

# Kerberos relay

[NTLM relay](../ntlm/relay.md) works because NTLM is not bound to a target. Kerberos is harder to relay because a service ticket is **encrypted for a specific SPN**, so you cannot simply forward one authentication to an arbitrary service. **krbrelayx** solves this two ways: it captures TGTs via **unconstrained delegation**, and, where you can make a victim authenticate to a **service name you control**, it relays that Kerberos authentication onward.

## Unconstrained delegation capture

The original krbrelayx mode is the companion to [unconstrained delegation](delegation/unconstrained.md): a host trusted for unconstrained delegation receives callers' **TGTs**, and krbrelayx, armed with that host's Kerberos key, decrypts the incoming authentication and extracts the TGT for reuse:

```bash
# Provide the unconstrained-delegation host's AES key or hash; krbrelayx extracts coerced TGTs
krbrelayx.py -aesKey <host-aes-key>
# then coerce a DC to authenticate to this host and capture DC$'s TGT
```

## Relaying to a controlled service name

To relay Kerberos (rather than capture a TGT), the victim must request a service ticket for an **SPN that resolves to you**. You arrange that by controlling name resolution and the SPN:

- Spoof a name via [ADIDNS](../ntlm/adidns.md) or `mitm6` so the victim's target resolves to your host, and coerce it; the victim requests a ticket for that host's SPN, which you relay.
- Relay the Kerberos authentication to **LDAP** (for RBCD or shadow credentials) or to **AD CS** web enrolment (ESC8):

```bash
# Relay coerced Kerberos auth to AD CS web enrolment for a certificate
krbrelayx.py --target 'http://<ca>/certsrv/' --adcs --template Machine
```

## Exploitation notes

- Kerberos relay sidesteps NTLM-hardening: where NTLM is disabled or SMB/LDAP signing blocks NTLM relay, a Kerberos path may still reach LDAP or AD CS.
- The constraint is the **SPN**: you need the victim to authenticate to a name you control, so it pairs tightly with [ADIDNS](../ntlm/adidns.md)/`mitm6` and [coercion](../ntlm/coercion.md).
- Unconstrained-delegation capture remains the most reliable krbrelayx use: coerce a DC to a delegation host you own and take `DC$`'s TGT, then DCSync.

## Tools

- **krbrelayx.py** (dirkjanm): TGT capture via unconstrained delegation and Kerberos relay to HTTP/LDAP/AD CS.
- **dnstool.py / mitm6**: control name resolution so the victim targets your SPN.
- **Coercer / PetitPotam**: source the authentication.

## References

- [krbrelayx (dirkjanm)](https://github.com/dirkjanm/krbrelayx)
- [dirkjanm: relaying Kerberos over DNS with krbrelayx and mitm6](https://dirkjanm.io/relaying-kerberos-over-dns-with-krbrelayx-and-mitm6/)
- [Synacktiv: relaying Kerberos over SMB using krbrelayx](https://www.synacktiv.com/en/publications/relaying-kerberos-over-smb-using-krbrelayx)
