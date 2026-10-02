---
title: "Group Policy: code execution and configuration across the domain"
description: "Active Directory Group Policy as an attack surface: editing a GPO you can write, linking one to an OU you control, and recovering the credentials left in SYSVOL, to run code on every computer and user the policy applies to."
keywords:
  - group policy
  - GPO abuse
  - SYSVOL
  - gPLink
  - scheduled task
---

# Group Policy

Group Policy applies configuration and scripts from the domain to the computers and users in its scope. A Group Policy Object (GPO) is a directory object plus a folder in SYSVOL, and it is enforced automatically by every machine it is linked to. That makes a GPO a delivery mechanism for code and configuration across many hosts at once: **write access to a GPO, or the ability to link one to an organizational unit, is remote code execution on everything in scope.**

## Why GPOs are a powerful target

- A single linked GPO can touch **every computer in an OU or the whole domain**, so one write escalates widely in one step.
- GPOs can deploy **scheduled tasks, startup/logon scripts, services, and registry changes**, all of which are direct execution or privilege primitives.
- The right to **link** a GPO to an OU (the `gPLink` attribute) is as powerful as editing one: link an attacker-controlled GPO to a target OU and it applies.
- Enforcement is automatic on the policy refresh cycle, so abuse needs no further interaction.

## How it is abused

- **Edit a writable GPO** to add an immediate scheduled task or script that runs as SYSTEM on in-scope machines.
- **Link a malicious GPO** to an OU you can write the `gPLink` on, bringing in-scope objects under its control.
- **Target the right scope**: a GPO linked to the Domain Controllers OU, or to a tier-0 OU, converts GPO control into domain compromise.

## Pages

- **[GPO and OU enumeration](gpo-and-ou-enumeration.md)**: finding writable GPOs, their links, and the OUs you can affect.
- **[Editing a GPO](editing-a-gpo.md)**: immediate scheduled tasks, scripts, and local admin on everything in scope.
- **[Linking a GPO](linking-a-gpo.md)**: `WriteGPLink` over an OU to bring its objects into a policy.
- **[Group Policy Preferences](group-policy-preferences.md)**: recovering `cpassword` and autologon credentials from SYSVOL.

## Tools

- **pyGPOAbuse** (Linux) / **SharpGPOAbuse** (Windows): inject immediate tasks, scripts, and local-admin changes into a writable GPO.
- **NetExec (`nxc`)**: `ldap --gpo` to list GPOs, `-M gpp_password` / `-M gpp_autologin` for SYSVOL credentials.
- **PowerView** (`New-GPOImmediateTask`, `Set-DomainObject` for `gPLink`), **Group3r / Grouper2**: GPO edits and vulnerable-setting audits.
- **bloodyAD**: set an OU's `gPLink` from Linux.

## References

- The Hacker Recipes: Group policies
- Microsoft: Group Policy and SYSVOL
