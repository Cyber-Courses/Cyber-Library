---
title: "Azure"
description: "Attacking the Microsoft Azure resource and management plane: Azure RBAC and managed-identity privilege escalation, credential theft from IMDS and Key Vault, and abuse of the compute, storage, serverless, data, networking, logging, and messaging services in a subscription."
keywords:
  - Azure
  - ARM
  - Azure RBAC
  - managed identity
  - subscription
---

# Azure

Azure is driven through the Azure Resource Manager (**ARM**) API, authorized by **Azure RBAC** role assignments and reached with a user token, a service-principal secret, or a resource's **managed identity**. Almost every attack is an RBAC question: what scope does my principal hold, which role lets it escalate, and which resource's managed identity can it borrow. The surfaces below follow that shape, with **privilege escalation** living inside [identity](identity/index.md) and credential theft inside [credentials](credentials/index.md).

This area is the **resource and management plane only**. Microsoft Entra ID (the tenant directory, formerly Azure AD) is attacked like a directory and lives under [Entra ID](../../directory/entra-id/index.md) next to Active Directory; user and application identity attacks (device code, PRT, consent, Entra roles) are there, not here.

## Enumeration

Enumeration folds into each surface, but the subscription-wide inventory is run first: `az account list` and `az resource list` for the resource graph, **Azure Resource Graph** queries for scale, role assignments with `az role assignment list --all`, and managed identities with **ROADtools**, **Stormspotter**, or **AzureHound** (BARK) for the RBAC and resource graph. Those feed every surface below.

## Surfaces

- **[Identity](identity/index.md)**: Azure RBAC role and custom-role writes, elevate-access, managed-identity assignment, and PIM activation.
- **[Credentials](credentials/index.md)**: managed-identity tokens from IMDS, Key Vault, storage keys, Automation assets, and app settings.
- **[Compute](compute/index.md)**: VM run command and extensions, AKS, Container Registry, Container Instances, and scale sets.
- **[Storage](storage/index.md)**: blob enumeration and access, SAS tokens, disk snapshots, and file shares.
- **[Serverless](serverless/index.md)**: Functions, Logic Apps, Automation Accounts, App Service, and Deployment Scripts.
- **[Data](data/index.md)**: SQL Database, Cosmos DB, Data Factory, Synapse, and storage tables.
- **[Networking](networking/index.md)**: network security groups and VNet, private endpoints, DNS takeover, Front Door, and Bastion.
- **[Logging and detection](logging-and-detection/index.md)**: tampering with the Activity Log, Azure Monitor, Defender for Cloud, and Sentinel.
- **[Messaging](messaging/index.md)**: Service Bus, Event Hubs, Event Grid, and Storage Queues.

## References

- [HackTricks Cloud: Azure](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI: MicroBurst](https://github.com/NetSPI/MicroBurst)
- [SpecterOps: BARK and AzureHound](https://github.com/BloodHoundAD/BARK)
- [ROADtools (Dirk-jan Mollema)](https://github.com/dirkjanm/ROADtools)
