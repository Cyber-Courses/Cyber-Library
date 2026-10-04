---
title: "Execution role: stealing a Lambda's credentials from the runtime"
description: "Stealing a Lambda function's execution-role credentials from the runtime environment variables and metadata endpoint."
keywords:
  - Lambda
  - execution role
  - environment variables
  - credentials
  - AWS_SESSION_TOKEN
---

# Execution role

Every Lambda function assumes an **execution role** and the runtime exposes that role's temporary credentials to the function process as environment variables. Any code execution inside the function, whether through a [code overwrite](code-and-layers.md), a dependency, or an application injection, can read them and continue as the role. Separately, `lambda:GetFunctionConfiguration` leaks the function's own environment variables from the API, which routinely hold database passwords and API keys.

## Reading credentials from inside the function

```bash
# the execution role's temporary credentials are injected as env vars
env | grep -E 'AWS_ACCESS_KEY_ID|AWS_SECRET_ACCESS_KEY|AWS_SESSION_TOKEN|AWS_REGION'
# AWS_LAMBDA_RUNTIME_API serves the same credentials to the runtime
```

Export those three values locally and you are the execution role.

## Reading config and secrets from the API

```bash
aws lambda get-function-configuration --function-name <fn> \
  --query 'Environment.Variables'
aws lambda list-functions \
  --query 'Functions[].[FunctionName,Role,Environment.Variables]'
```

## Exploitation notes

- The injected credentials are the execution role's; their power is whatever that role holds, so enumerate it with the stolen keys.
- `get-function-configuration` returns environment variables in plaintext unless they were encrypted with a customer KMS key you cannot use; unencrypted secrets drop straight out.
- To obtain code execution where you have no injection, overwrite the function with [code and layers](code-and-layers.md).

## Tools

- **AWS CLI** (`lambda get-function-configuration`, `list-functions`): config and secret enumeration.
- **Pacu** (`lambda__enum`): bulk function and environment-variable enumeration.

## References

- [HackTricks Cloud: AWS Lambda](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: Lambda runtime environment variables](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Rhino Security Labs: AWS privilege escalation](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
