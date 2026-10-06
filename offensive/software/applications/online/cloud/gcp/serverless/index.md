---
title: "GCP serverless"
order: 5
description: "Attacking GCP serverless and CI: Cloud Functions and Cloud Run service accounts, and Cloud Build service-account and trigger abuse."
keywords:
  - Cloud Functions
  - Cloud Run
  - Cloud Build
  - service account
  - CI/CD
---

# Serverless

GCP's serverless and CI services each run as a **service account**, so they are both a loot target (read their source, environment, and secrets) and an execution surface (deploy or trigger code that runs as a privileged account). Cloud Build is the standout: its default service account historically carries project **Editor**, so a single build you control mints a near-admin token.

The deploy-as-service-account privilege-escalation angle (`iam.serviceAccounts.actAs` plus a create verb) is cataloged under [identity privilege escalation](../identity/privilege-escalation/index.md); the pages here cover attacking the services themselves and using them for persistence.

## Pages

- **[Cloud Functions](cloud-functions.md)**: reading source and environment, unauthenticated invocation, and the runtime service account.
- **[Cloud Run](cloud-run.md)**: unauthenticated services and jobs, revision secrets, and the attached service account.
- **[Cloud Build](cloud-build/index.md)**: exfiltrating the build service account token and planting build triggers for persistence.

## References

- [HackTricks Cloud: GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Rhino Security Labs: GCP privilege escalation](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
