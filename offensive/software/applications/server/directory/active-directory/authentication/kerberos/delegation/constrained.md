---
title: "Constrained delegation: S4U and protocol transition"
description: "Abusing Kerberos constrained delegation: using S4U2self and S4U2proxy from a compromised delegating account to obtain service tickets as any user, including the protocol-transition case that needs no prior user authentication."
keywords:
  - constrained delegation
  - S4U2self
  - S4U2proxy
  - protocol transition
  - msDS-AllowedToDelegateTo
---

# Constrained delegation

Constrained delegation lets an account request service tickets for a user, but only to a **fixed list of SPNs** named in its `msDS-AllowedToDelegateTo` attribute. If you compromise such an account (its password, hash, or AES key), you can impersonate arbitrary users to those listed services, and often to more than the list literally allows.

## The abuse with protocol transition

When the account is configured for **protocol transition** (`TRUSTED_TO_AUTH_FOR_DELEGATION`), it can call **S4U2self** to mint a ticket to itself as *any* user with no involvement from that user, then **S4U2proxy** to turn it into a ticket for the allowed back-end service as that user:

```bash
# Impacket: impersonate Administrator to an allowed SPN, using the delegating account's hash
getST.py -spn cifs/target.example.local -impersonate Administrator \
  -hashes :<delegating-acct-hash> example.local/delegating-acct
export KRB5CCNAME=Administrator.ccache
```

Without protocol transition, S4U2self still works but the resulting ticket is not forwardable, so S4U2proxy is supposed to refuse it. In practice the ticket can often still be used, and self/U2U tricks can produce a usable one.

## Bronze Bit: forcing the forwardable flag

The **Bronze Bit** attack removes the two constraints above. When you hold the delegating service's key, you can take the S4U2self ticket, **flip its `forwardable` flag yourself** (decrypt with the service key, set the flag, re-encrypt), and submit it to S4U2proxy. The KDC accepts it, so delegation succeeds **without protocol transition**, and crucially it also lets you impersonate accounts that are supposed to be undelegatable: members of **Protected Users** and accounts marked **"sensitive and cannot be delegated"**. Microsoft constrained this with a November 2020 update (the KDC now validates the flag), so it applies to unpatched DCs.

```bash
# Impacket getST implements it with -force-forwardable
getST.py -spn cifs/target.example.local -impersonate Administrator -force-forwardable \
  -hashes :<delegating-acct-hash> example.local/delegating-acct
```

## The SPN-substitution trick

S4U2proxy constrains the *service class* loosely: the returned ticket's **service name** can be swapped to another SPN (the KDC does not bind the ticket tightly to the exact SPN string). The catch is the key: the ticket is encrypted with the back-end account's key, so the substituted SPN only works if it is **registered to that same account**. In practice a host's own service SPNs (`cifs/`, `host/`, `http/`, `ldap/` on a DC, and so on) are all backed by the **computer account**, so delegation allowed to one of them can be swapped to the others on that host, turning a limited-looking delegation into full control of that machine. It does **not** reach an SPN registered to a *different*, dedicated service account, because that account's key cannot decrypt the ticket.

## Exploitation notes

- Impersonate a privileged user (Domain Admin) to a `cifs/` or `host/` SPN on the target to get file access or code execution as that user on that host.
- The SPN-substitution trick means the listed SPN matters less than the **target account**: delegation to one computer-account SPN is effectively delegation to all of that machine's own services, though not to SPNs held by separate service accounts on the same host.
- You need the delegating account's secret; recover it by [cracking](../../credentials/cracking.md) (it is often a service account) or dumping, then drive S4U with its hash or AES key.
- Accounts marked "sensitive, cannot be delegated" and members of Protected Users cannot be impersonated this way.

## Tools

- **Impacket** (`getST.py` with `-impersonate`): S4U2self/S4U2proxy and SPN substitution.
- **Rubeus** (`s4u`): on-host constrained-delegation abuse.
- **findDelegation.py** (Impacket): enumerate accounts configured for delegation.

## References

- [Elad Shamir: Wagging the Dog](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
- [Impacket getST (-impersonate)](https://github.com/fortra/impacket)
- [GhostPack Rubeus (s4u)](https://github.com/GhostPack/Rubeus)
