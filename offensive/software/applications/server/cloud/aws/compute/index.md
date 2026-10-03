---
title: "AWS compute"
description: "Abusing AWS compute services: EC2 instance user-data and role credentials, remote command execution through SSM, EBS snapshot theft, and the container services ECS and EKS as paths to credentials and code execution."
keywords:
  - EC2
  - SSM
  - user-data
  - EBS snapshot
  - ECS
---

# Compute

Compute instances run with an **instance role**, hold **user-data** that often contains secrets, and can frequently be driven remotely through **SSM**, so compute access is both a credential source and a code-execution surface. The role an instance carries is usually the quickest pivot from a single box to the account.

The pages here cover reading and abusing **user-data** and the instance role (via [IMDS](../credentials/index.md)), running commands on instances with **`ssm:SendCommand`** / Run Command, stealing data by **sharing or mounting EBS snapshots** into an account you control, and reaching workloads through **ECS** task roles and **EKS** (cross-referenced to the container internals in the Containers area).

## What folds in here

- **Code execution** on instances (SSM, user-data) and the credential theft that follows.
- **Persistence** on compute (modified user-data, launch templates) is noted here.
- **EKS/container** specifics beyond the AWS control plane are cross-referenced, not duplicated.

## References

- [HackTricks Cloud: AWS EC2, SSM, and compute](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: Systems Manager Run Command](https://docs.aws.amazon.com/systems-manager/latest/userguide/run-command.html)
