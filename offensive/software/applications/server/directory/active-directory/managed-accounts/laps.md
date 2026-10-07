---
title: "LAPS: reading local-administrator passwords from the directory"
order: 2
description: "Recovering machine local-administrator passwords that LAPS stores in Active Directory: the legacy ms-Mcs-AdmPwd cleartext attribute and the Windows LAPS msLAPS-Password / msLAPS-EncryptedPassword attributes, read over LDAP with a sufficient ACE."
keywords:
  - LAPS
  - ms-Mcs-AdmPwd
  - msLAPS-Password
  - ReadLAPSPassword
  - local administrator
---

# LAPS

The Local Administrator Password Solution (LAPS) randomises each machine's local-administrator password and stores it **in the computer's directory object**. Convenient, and directly abusable: a read right over the right attribute hands you that host's local admin password. BloodHound draws it as the `ReadLAPSPassword` edge. There are two generations, stored in different attributes.

## Legacy LAPS vs Windows LAPS

- **Legacy LAPS**: the password is **cleartext** in **`ms-Mcs-AdmPwd`** (expiry in `ms-Mcs-AdmPwdExpirationTime`). Any principal with read access to that attribute gets the password as-is.
- **Windows LAPS** (built into current Windows): uses **`msLAPS-Password`** (cleartext JSON when encryption is off) and **`msLAPS-EncryptedPassword`** (DPAPI-NG encrypted to a chosen principal/group), plus password **history** attributes. The encrypted form requires you to be (or reach) the authorised decryptor. BloodHound also tracks **`SyncLAPSPassword`**, a domain-level edge for retrieving confidential and RODC-filtered attributes (including the legacy `ms-Mcs-AdmPwd`) through directory synchronization, as distinct from **`ReadLAPSPassword`**, which is direct read access on a single computer object.

## Reading it

```bash
# NetExec: pull LAPS passwords over LDAP (handles legacy and Windows LAPS attributes)
nxc ldap <dc> -u user -p pass -M laps
nxc ldap <dc> -u user -p pass -M laps -o COMPUTER='WKSTN-*'   # filter to hosts

# pyLAPS: get (and set) the LAPS attribute from Linux
pyLAPS.py --action get -u user -p pass -d example.local --dc-ip <dc>

# bloodyAD: raw attribute read
bloodyAD --host <dc> -d example.local -u user -p pass get object 'WKSTN01$' --attr ms-Mcs-AdmPwd
```

## Exploitation notes

- A recovered LAPS password is **local admin on that one host** (LAPS gives each machine a unique password), so it is lateral movement one box at a time, not a master key, dump its secrets and pivot.
- Who can read LAPS is set by the OU's delegation; hunt for over-broad read grants (a helpdesk group, or `Authenticated Users` by misconfiguration) with [ACL enumeration](../dacl/acl-enumeration.md).
- Windows LAPS **encrypted** passwords need the authorised decryptor; if you only have read and not decryption rights, the blob is useless until you reach that principal.
- LAPS password **history** (Windows LAPS) can expose an older password still valid elsewhere or useful against cached material.

## Tools

- **NetExec (`nxc`) `-M laps`**: read legacy and Windows LAPS attributes over LDAP.
- **pyLAPS / LAPSDumper**: Linux get/set of the LAPS attributes.
- **bloodyAD** (`get object --attr`): raw read of `ms-Mcs-AdmPwd` / `msLAPS-Password`.

## References

- [SpecterOps: BloodHound ReadLAPSPassword edge](https://bloodhound.specterops.io/resources/edges/read-laps-password)
- [Microsoft: Windows LAPS overview](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview)
- [Microsoft (MS-ADA2): ms-LAPS-Password attribute](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-ada2/b2e01af2-3ff5-4c64-8ef3-d0d8a545945b)
- [NetExec LAPS module source](https://github.com/Pennyw0rth/NetExec/blob/main/nxc/modules/laps.py)
