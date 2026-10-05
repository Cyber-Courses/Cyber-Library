---
title: "Cloud Build"
description: "Abusing Cloud Build as a privilege-escalation and persistence surface: the build service account's broad roles and attacker-controlled build triggers."
keywords:
  - Cloud Build
  - build service account
  - triggers
  - CI/CD
  - privilege escalation
---

# Cloud Build

Cloud Build runs every build as the Cloud Build **service account** (`<project-number>@cloudbuild.gserviceaccount.com`), which by default has historically been granted the project **Editor** role. Anyone who can start a build, or plant a trigger that starts one, therefore runs arbitrary steps as a near-admin principal. This makes Cloud Build one of the strongest GCP privilege-escalation and persistence surfaces.

## Pages

- **[Build service account](build-service-account.md)**: starting a build whose steps exfiltrate the build SA token and act with its roles.
- **[Build triggers](build-triggers.md)**: planting or editing repository triggers so pushes run attacker steps as the build SA.

## References

- [Rhino Security Labs: GCP privilege escalation (Cloud Build)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [HackTricks Cloud: GCP Cloud Build](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
