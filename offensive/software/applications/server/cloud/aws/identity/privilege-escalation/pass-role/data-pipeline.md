---
title: "Data Pipeline: PassRole for shell execution on provisioned resources"
description: "iam:PassRole into AWS Data Pipeline to run arbitrary shell commands on provisioned resources under a privileged resource role."
keywords:
  - PassRole
  - Data Pipeline
  - resource role
  - command execution
  - EC2
---

# Data Pipeline

AWS Data Pipeline provisions compute (an EC2 resource) to run pipeline activities under a **resource role**. With `iam:PassRole` and `datapipeline:CreatePipeline` plus `PutPipelineDefinition`, you define a `ShellCommandActivity` that runs your commands on that resource, which carries the passed role's credentials through its instance profile.

## Define a shell activity

```bash
aws datapipeline create-pipeline --name x --unique-id x
aws datapipeline put-pipeline-definition --pipeline-id <id> \
  --pipeline-objects file://objects.json \
  --parameter-values myRole=<privileged-resource-role>
aws datapipeline activate-pipeline --pipeline-id <id>
```

The `ShellCommandActivity` in `objects.json` runs on the provisioned EC2 resource, so it can query IMDS for the resource role's credentials and exfiltrate them.

## Exploitation notes

- Data Pipeline is a legacy service but remains a known escalation when the resource role is privileged.
- The activity runs on a real instance, so the [EC2 IMDS](../../../credentials/instance-metadata/index.md) retrieval applies once the command lands.

## Tools

- **AWS CLI** (`datapipeline create-pipeline` / `put-pipeline-definition`).
- **Pacu**: Data Pipeline privesc module.

## References

- [Rhino Security Labs: AWS privilege escalation (Data Pipeline)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
