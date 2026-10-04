---
title: "Data Factory: pipelines and linked services as the factory identity"
description: "Abusing Azure Data Factory: pipelines and linked services that run as the factory's managed identity, and stored connection secrets."
keywords:
  - Data Factory
  - pipeline
  - linked service
  - managed identity
  - integration runtime
---

# Data Factory

Azure Data Factory runs data pipelines, and a pipeline executes as the factory's **managed identity**. A principal that can author and trigger a pipeline runs activities (Web, Azure Function, or a custom activity) under that identity, which turns control of a data factory into a token for whatever the factory identity is granted. Linked services also store connection secrets that can be read back.

## Running a pipeline as the factory identity

```bash
az datafactory list -g <rg> -o table
# author a pipeline with a Web activity that calls ARM or Graph as the factory MI,
# or that hits IMDS-style token endpoints, then trigger it
az datafactory pipeline create-run --factory-name <df> -g <rg> --name <pipeline>
```

## Looting linked services

```bash
# linked services hold connection strings and credential references
az datafactory linked-service list --factory-name <df> -g <rg>
```

## Exploitation notes

- The factory system-assigned identity is frequently granted Storage or Key Vault access so pipelines can read data; a Web activity calling those APIs inherits that access.
- A self-hosted integration runtime runs on a VM you may be able to reach, and holds credentials for on-prem sources.
- Pipeline runs are a quiet execution channel: they look like normal data movement, not interactive compute.

## Tools

- **az cli** (`az datafactory pipeline create-run`, `az datafactory linked-service list`).
- **MicroBurst**: enumerates data factories and linked-service secrets.

## References

- [Microsoft: Data Factory managed identity](https://learn.microsoft.com/azure/data-factory/data-factory-service-identity)
- [HackTricks Cloud: Azure Data Factory](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [SpecterOps: Azure managed identity attack paths (BARK)](https://github.com/BloodHoundAD/BARK)
