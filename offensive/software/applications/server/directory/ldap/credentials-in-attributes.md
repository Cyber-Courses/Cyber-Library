---
title: "Credentials in attributes: passwords and secrets stored in directory objects"
description: "Harvesting credentials that administrators and the directory itself store in LDAP object attributes: description and info fields, userPassword, and Active Directory LAPS and gMSA managed-password attributes."
keywords:
  - credentials in LDAP
  - description field
  - userPassword
  - LAPS
  - gMSA
---

# Credentials in attributes

Directories end up holding plaintext and recoverable credentials in object attributes, partly from administrators using fields as notes and partly from features that store machine and service passwords in the directory for authorized principals to read. Reading the right attributes turns directory access into credential access.

## Admin-stored secrets

The classic leak is a password typed into a free-text field:

```bash
# Grep the directory for password-like content in notes fields
ldapsearch ... "(|(description=*pass*)(description=*pwd*)(info=*))" sAMAccountName description info

# Broad sweep for any non-empty description across users
ldapsearch ... "(&(objectClass=user)(description=*))" sAMAccountName description
```

`description`, `info`, `comment`, and custom schema fields frequently contain onboarding passwords, service-account passwords, and "temporary" credentials that were never removed.

## userPassword

On non-AD directories (OpenLDAP, 389 DS), the `userPassword` attribute stores the account's password, often as a hash (`{SSHA}`, `{MD5}`) and occasionally in cleartext. Where ACLs are misconfigured, it is readable by a bound user or even anonymously, and the hashes crack offline:

```bash
ldapsearch ... "(objectClass=person)" userPassword
```

In AD, `userPassword` is usually not populated or not readable, but `unixUserPassword` and similar attributes appear in environments with identity-management integration.

## Active Directory LAPS and gMSA

AD stores machine and service secrets in the directory for principals granted read access:

- **LAPS**: the local administrator password of a domain-joined machine is stored in `ms-MCS-AdmPwd` (legacy LAPS) or `msLAPS-Password` (Windows LAPS). Any principal with the read right (often a helpdesk group) reads the local admin password straight from the computer object.

  ```bash
  nxc ldap <dc> -u user -p pass -M laps
  ldapsearch ... "(ms-MCS-AdmPwd=*)" ms-MCS-AdmPwd sAMAccountName
  ```

- **gMSA**: a group managed service account's password blob is in `msDS-ManagedPassword`, readable by the principals listed in `msDS-GroupMSAMembership`. Tools compute the NT hash from the returned blob for pass-the-hash.

  ```bash
  nxc ldap <dc> -u user -p pass --gmsa
  gMSADumper.py -u user -p pass -d example.local
  ```

Who can read these is an [ACL](../active-directory/enumeration/acl-enumeration.md) question, so enumerating the read rights on LAPS/gMSA-bearing objects tells you which secrets are within reach.

## Exploitation notes

- Sweep `description`/`info` early; it is the single highest-yield, zero-privilege credential source in many domains.
- A recovered LAPS password is local admin on that one machine (LAPS randomizes per host), which is still a foothold for credential dumping and session hunting.
- gMSA and machine-account hashes are strong (long and random) so they do not crack, but they are directly usable for pass-the-hash and, for machine accounts, for RBCD and S4U abuse.

## Tools

- **ldapsearch**: attribute sweeps for notes fields and `userPassword`.
- **NetExec (nxc) ldap -M laps / --gmsa**: LAPS and gMSA extraction.
- **gMSADumper.py**: compute gMSA NT hashes from the managed-password blob.

## References

- The Hacker Recipes: credential dumping (directory-stored secrets)
- Microsoft: LAPS and group managed service accounts
