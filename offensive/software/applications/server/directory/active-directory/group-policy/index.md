---
title: "Group Policy: abusing GPOs for code execution and configuration change"
description: "Abusing Active Directory Group Policy: editing a GPO you can write, or linking one to an OU you control, to run code and change security configuration on every computer and user the policy applies to."
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

## References

- The Hacker Recipes: GPO abuse
- Microsoft: Group Policy and SYSVOL
