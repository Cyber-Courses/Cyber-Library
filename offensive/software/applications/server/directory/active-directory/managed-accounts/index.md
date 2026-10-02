---
title: "Managed accounts: directory-stored account secrets and dMSA abuse"
description: "Abusing the accounts and passwords Active Directory manages for you: reading group Managed Service Account (gMSA) passwords and LAPS local-administrator passwords from the directory, and the BadSuccessor delegated-MSA privilege escalation on Windows Server 2025."
keywords:
  - gMSA
  - LAPS
  - dMSA
  - BadSuccessor
  - managed service account
---

# Managed accounts

Active Directory can **manage account secrets for you**: group Managed Service Accounts (gMSA) have their passwords generated and stored in the directory, LAPS keeps each machine's local-administrator password in the directory, and Windows Server 2025 adds delegated Managed Service Accounts (dMSA). Convenient for administrators, and a gift for an attacker: the secret lives in an attribute, so a read right hands you the password, and the dMSA migration mechanism can be abused to impersonate any account outright.

## The surfaces

- **gMSA**: the managed password is in `msDS-ManagedPassword`, readable only by the principals in `msDS-GroupMSAMembership` (`PrincipalsAllowedToRetrieveManagedPassword`). Land in that set and you compute the account's NT hash straight from the directory.
- **LAPS**: the per-machine local-admin password is in `ms-Mcs-AdmPwd` (legacy, cleartext) or the `msLAPS-*` attributes (Windows LAPS, optionally encrypted). A read right over the computer's attribute is local admin on that host.
- **dMSA (BadSuccessor)**: on a Server 2025 domain, writing a dMSA's migration attributes makes the KDC build its PAC from a **superseded** account's SIDs, so a low-privileged create/write right becomes impersonation of any account, including Domain Admin.

## What ties them together

All three are **directory-driven**: the secret or the trust decision lives in an object's attributes, so the attack is an LDAP read or write, not code on a host. That makes them quiet, Linux-friendly, and dependent only on the right ACE, which is why they belong with [DACL](../dacl/index.md) enumeration: BloodHound's `ReadGMSAPassword` and `ReadLAPSPassword` edges point straight at them.

## Pages

- **[gMSA](gmsa.md)**: reading `msDS-ManagedPassword` to recover a service account's key.
- **[LAPS](laps.md)**: reading local-administrator passwords from the directory.
- **[BadSuccessor](badsuccessor.md)**: dMSA migration abuse for privilege escalation on Server 2025.

## References

- The Hacker Recipes: ReadGMSAPassword, ReadLAPSPassword
- Akamai: BadSuccessor (dMSA privilege escalation)
