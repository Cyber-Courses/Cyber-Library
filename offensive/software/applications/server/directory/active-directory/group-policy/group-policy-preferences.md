---
title: "Group Policy Preferences: credentials in SYSVOL"
description: "Recovering credentials from Group Policy Preferences XML left in SYSVOL: the cpassword field encrypted with Microsoft's published AES key, and autologon credentials, both readable by any domain user and decryptable offline."
keywords:
  - group policy preferences
  - cpassword
  - SYSVOL
  - gpp_password
  - MS14-025
---

# Preferences

Group Policy Preferences (GPP) let administrators push local users, scheduled tasks, mapped drives, and services to machines, and older ones store a password in the policy XML as a **`cpassword`** field. That field is AES-encrypted with a key Microsoft **published**, so anyone who can read the XML can decrypt it. Because the policy files live in **SYSVOL**, which every authenticated domain user can read, a single `cpassword` is a domain-user-to-local-admin (and often reused-everywhere) credential handed out for free.

## Finding and decrypting

```bash
# NetExec: locate and decrypt cpassword across SYSVOL in one step
nxc smb <dc> -u user -p pass -M gpp_password
# Autologon credentials stored in Registry.xml preferences
nxc smb <dc> -u user -p pass -M gpp_autologin

# Manual: grep SYSVOL for cpassword, then decrypt with the published key
grep -rl 'cpassword' /mnt/sysvol/   # Groups.xml, Services.xml, Drives.xml, ScheduledTasks.xml, Datasources.xml
gpp-decrypt '<cpassword-blob>'
```

```powershell
# On a domain host
Get-GPPPassword
```

The XML files that carry `cpassword` are `Groups.xml` (local users), `Services.xml`, `ScheduledTasks.xml`, `Drives.xml`, `Datasources.xml`, and `Printers.xml` under each policy's `Machine` or `User` preferences folder.

## Why it still matters

- Microsoft removed the ability to **create** new GPP passwords in 2014 (MS14-025), but it did **not** scrub existing ones, so legacy `cpassword` entries persist in SYSVOL for years.
- The recovered account is frequently a **local administrator** pushed to many machines with the same password, so one find enables [pass-the-hash](../authentication/ntlm/pass-the-hash.md)/password reuse across the fleet.
- It needs only a single low-privileged domain account (SYSVOL read), making it a first-move credential hunt alongside [reconnaissance](../authentication/credentials/index.md).

## Exploitation notes

- Grep the **whole** SYSVOL policy tree, not just `Groups.xml`; credentials hide in services, scheduled tasks, and datasource preferences too.
- Autologon credentials (`gpp_autologin`, from `Registry.xml`) are cleartext, not `cpassword`-encrypted, and often expose a privileged account set to log a kiosk or build machine in automatically.
- Validate and spray the recovered credential with `nxc` to map where that local account is reused.

## Tools

- **NetExec (`nxc`) `-M gpp_password` / `-M gpp_autologin`**: find and decrypt from Linux.
- **gpp-decrypt**: decrypt a `cpassword` blob with the published AES key.
- **PowerSploit `Get-GPPPassword`**: on-host recovery.

## References

- The Hacker Recipes: Group Policy Preferences passwords
- Microsoft: MS14-025 (Group Policy Preferences password removal)
