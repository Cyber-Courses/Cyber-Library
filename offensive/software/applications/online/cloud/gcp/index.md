---
title: "GCP"
description: "Attacking Google Cloud Platform: enumerating and abusing Cloud IAM, impersonating service accounts, harvesting credentials from metadata and Secret Manager, and exploiting compute, storage, serverless, data, networking, logging, and Pub/Sub across projects and the organization."
keywords:
  - GCP
  - Cloud IAM
  - service account
  - actAs
  - project
---

# GCP

Google Cloud is driven through the Cloud APIs, authorized by **Cloud IAM** bindings and reached with a user token, a **service-account** key, or a resource's attached service account. Almost every attack is an IAM question: which members hold which roles at which scope (organization, folder, project, or resource), which permission lets you **impersonate** or **actAs** a more privileged service account, and which resource you can deploy to run as one. The surfaces below follow that shape, with **privilege escalation** living inside [identity](identity/index.md).

## Enumeration

Enumeration folds into each surface, but the project- and organization-wide inventory is run first: `gcloud projects list` and `gcloud asset search-all-resources` for the resource graph, `gcloud projects get-iam-policy` for the bindings, and **GCPBucketBrute**, **gcloud**, and the IAM graph tooling to resolve who can reach which service account. Those feed every surface below.

## Surfaces

- **[Identity](identity/index.md)**: Cloud IAM `setIamPolicy`, the `actAs` deploy-as-service-account catalog, metadata, service-account impersonation, and workload identity federation.
- **[Credentials](credentials/index.md)**: the metadata server, Secret Manager, service-account keys, gcloud and ADC tokens, Cloud KMS, HMAC keys, and API keys.
- **[Compute](compute/index.md)**: Compute Engine metadata, SSH, and service-account scopes, and GKE cluster access and node identity.
- **[Storage](storage/index.md)**: Cloud Storage bucket enumeration and access, disk snapshots, and Filestore.
- **[Serverless](serverless/index.md)**: Cloud Functions, Cloud Run, and Cloud Build service-account and trigger abuse.
- **[Data](data/index.md)**: Cloud SQL, BigQuery, Vertex AI, Dataproc, Dataflow, Firestore, Spanner, and Bigtable.
- **[Networking](networking/index.md)**: firewall-rule exposure, VPC reach, Cloud DNS and subdomain takeover, and load balancing.
- **[Logging and detection](logging-and-detection/index.md)**: tampering with Cloud Logging sinks, disabling data-access audit logs, and weakening Security Command Center.
- **[Messaging](messaging/index.md)**: Pub/Sub topics and subscriptions, and Cloud Tasks queues.

## References

- [HackTricks Cloud: GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Rhino Security Labs: GCP privilege escalation](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-1/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
