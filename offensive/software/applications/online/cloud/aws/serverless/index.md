---
title: "AWS serverless"
description: "Attacking AWS serverless: Lambda execution roles, code and layers, API Gateway resource policies and authorizer bypass, Function URLs, Step Functions, and EventBridge."
keywords:
  - Lambda
  - execution role
  - API Gateway
  - Function URL
  - serverless
---

# Serverless

A serverless component runs with a role and holds configuration that frequently carries secrets, so reaching one is reaching its **role** and its **config**. Lambda is the center of gravity: a function runs under an **execution role**, its environment variables often hold credentials, and its code is attacker-replaceable. Around it, API Gateway fronts functions and other AWS services, Function URLs expose them directly, and Step Functions and EventBridge orchestrate privileged actions under their own roles.

## What folds in here

- **[Execution role](execution-role.md)**: stealing a Lambda function's role credentials from the runtime.
- **[Code and layers](code-and-layers.md)**: reading source and layers for secrets, and overwriting them to run under the role.
- **[API Gateway](api-gateway/index.md)**: resource-policy exposure, authorizer bypass, and the integration role.
- **[Function URL](function-url.md)**: a Lambda Function URL with lax auth, invoked directly over HTTPS.
- **[Step Functions](step-functions.md)**: state machines and their execution role.
- **[EventBridge](eventbridge.md)**: rules, buses, and targets fired under a privileged role.

Creating or updating a function to assume a more powerful role is the [iam:PassRole](../identity/privilege-escalation/pass-role/lambda.md) path and lives under identity; the pages here cover abusing the services themselves.

## References

- [HackTricks Cloud: AWS Lambda](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: Lambda execution role](https://docs.aws.amazon.com/lambda/latest/dg/lambda-intro-execution-role.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Rhino Security Labs: AWS privilege escalation](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
