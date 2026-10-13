---
title: "Enumeration: mapping the IAM graph and reachable service accounts"
order: 1
description: "Mapping a GCP project's IAM bindings, members, custom roles, and service accounts from a foothold: getIamPolicy calls, gcloud, and tooling like ScoutSuite to find reachable identities and permissions."
keywords:
  - IAM enumeration
  - getIamPolicy
  - gcloud
  - service accounts
  - custom roles
  - ScoutSuite
---

# Enumeration

Every escalation in GCP starts from the IAM graph: which members hold which roles at which scope, and which service accounts you can reach by impersonation or `actAs`. The answers come from `getIamPolicy` calls at each level and from resolving the permissions your own identity holds.

## Who am I and what do I hold

```bash
gcloud auth list                                    # active identities
gcloud config get-value project
gcloud projects get-iam-policy <project> --format=json   # all bindings at the project
# test a specific permission set against the project
gcloud iam list-testable-permissions //cloudresourcemanager.googleapis.com/projects/<project>
```

## Reading the bindings across scopes

```bash
gcloud organizations get-iam-policy <org-id>
gcloud resource-manager folders get-iam-policy <folder-id>
gcloud projects get-iam-policy <project>
# service accounts you might impersonate or actAs
gcloud iam service-accounts list
gcloud iam service-accounts get-iam-policy <sa-email>   # who can mint its token
```

## Resolving reachable paths

```bash
# asset inventory for the whole resource graph
gcloud asset search-all-resources --scope=projects/<project>
gcloud asset search-all-iam-policies --scope=projects/<project>
```

ScoutSuite and the IAM-graph scripts in the Rhino GCP-IAM-Privilege-Escalation repo turn this dump into a list of escalation edges: who can `setIamPolicy`, who holds `actAs` plus a deploy verb, and which service accounts have Token Creator bound to a principal you control.

## Exploitation notes

- `get-iam-policy` at a scope returns only the bindings set at that scope; roles inherited from a parent folder or the org are resolved with the asset-inventory policy search, not the project policy alone.
- A denied `getIamPolicy` does not mean you are blocked: `list-testable-permissions` and simply trying the escalation calls reveal the usable surface.
- The default Compute Engine and App Engine service accounts frequently hold Editor, so note every resource that runs as one.

## Tools

- **gcloud** (`get-iam-policy`, `asset search-all-iam-policies`, `list-testable-permissions`): the native graph dump.
- **ScoutSuite**: cross-service posture and IAM inventory.
- **GCP-IAM-Privilege-Escalation** (Rhino): scripts that enumerate permissions and flag escalation paths.

## References

- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: understanding IAM policies](https://cloud.google.com/iam/docs/policies)
