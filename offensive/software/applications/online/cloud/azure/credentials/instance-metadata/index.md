---
title: "Instance metadata"
order: 1
description: "Reading the Azure Instance Metadata Service to mint managed-identity tokens, including through SSRF against a victim resource."
keywords:
  - IMDS
  - instance metadata
  - managed identity token
  - SSRF
  - 169.254.169.254
---

# Instance metadata

The Azure Instance Metadata Service (IMDS) answers on the link-local `169.254.169.254` to any process on a VM, and its token endpoint hands out an **access token for the resource's managed identity**. Any foothold on a managed-identity-bearing resource, direct code execution or a server-side request forgery, turns into a token for whatever that identity can reach.

## What folds in here

- **[Managed identity token](managed-identity-token.md)**: requesting and replaying the token from the IMDS endpoint on the host.
- **[SSRF to IMDS](ssrf-to-imds.md)**: reaching the same endpoint through a vulnerable application.

Unlike AWS, Azure IMDS requires a `Metadata: true` request header and takes a `resource` parameter naming the audience (ARM, Microsoft Graph, Key Vault, storage), so a token is scoped to one audience at a time.

## References

- [Microsoft: acquire an access token from IMDS](https://learn.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token)
- [HackTricks Cloud: Azure IMDS](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
