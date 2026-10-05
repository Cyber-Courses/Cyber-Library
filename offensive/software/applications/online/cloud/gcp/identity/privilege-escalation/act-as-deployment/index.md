---
title: "actAs deployment"
description: "Deploying a resource that runs as a privileged service account via iam.serviceAccounts.actAs: creating a compute instance, function, Cloud Run service, build, or job bound to a higher-privileged account."
keywords:
  - actAs
  - service account
  - deploy
  - privilege escalation
  - compute.instances.create
  - Cloud Build
---

# actAs deployment

`iam.serviceAccounts.actAs` is the permission to bind a service account to a resource you create. On its own it does nothing; combined with a resource-create permission, it lets you deploy something that **runs as** a service account more privileged than you, then take its token or run code as it. The target service account only needs to exist and be usable in the project.

The pattern is always: find a privileged service account (`gcloud iam service-accounts list`, and the [enumeration](../../enumeration.md) graph for its role bindings), confirm you hold `actAs` on it, then drive it through one of the deploys below.

## Deploy targets

- **[Compute instance](compute-instance.md)**: `compute.instances.create` with a privileged service account and full scope.
- **[Cloud Function](cloud-function.md)**: deploy a function running as the attached account.
- **[Cloud Run](cloud-run.md)**: a service or job running attacker containers as the account.
- **[Deployment Manager](deployment-manager.md)**: deploy as the default-Editor Google APIs service agent.
- **[Cloud Build](cloud-build.md)**: exfiltrate the Cloud Build service account token mid-build.
- **[App Engine](app-engine.md)**: a version running as the App Engine default account.
- **[Cloud Composer](cloud-composer.md)**: an Airflow DAG task running as the environment account.
- **[Cloud Scheduler](cloud-scheduler.md)**: a scheduled job calling an API as a chosen account.
- **[Vertex AI notebook](vertex-ai-notebook.md)**: a notebook VM with the attached account.
- **[Dataproc](dataproc.md)**: a cluster or job running as the cluster account.

## References

- [Rhino Security Labs: GCP privilege escalation (part 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google: iam.serviceAccounts.actAs](https://cloud.google.com/iam/docs/service-accounts-actas)
