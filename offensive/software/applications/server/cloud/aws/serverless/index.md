---
title: "AWS serverless"
description: "Abusing AWS Lambda and serverless services: extracting the function execution role and environment secrets, tampering with function code and layers, and using over-permissioned functions as a privilege-escalation and persistence foothold."
keywords:
  - Lambda
  - execution role
  - environment variables
  - layers
  - serverless
---

# Serverless

A Lambda function runs with an **execution role** and carries **environment variables** that frequently hold secrets, so reaching a function is reaching its role and its config. Over-permissioned functions are also a privilege-escalation lever (through [iam:PassRole](../identity/index.md)) and a quiet persistence spot.

The pages here cover reading a function's **environment variables** and role credentials, modifying **function code or layers** to run attacker code with the role, invoking functions to reach internal resources, and planting a function or layer for **persistence**.

## What folds in here

- **Privilege escalation** via creating or updating a function that assumes a more powerful role is cross-referenced to [identity](../identity/index.md).
- **Persistence** through a deployed function or a poisoned shared layer.

## References

- [HackTricks Cloud: AWS Lambda](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: Lambda execution role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)
