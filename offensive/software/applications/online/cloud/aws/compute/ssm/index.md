---
title: "SSM"
order: 2
description: "Running commands on managed instances through Systems Manager: ssm:SendCommand and Session Manager shells under the instance's role."
keywords:
  - SSM
  - SendCommand
  - Session Manager
  - managed instance
  - command execution
---

# SSM

AWS Systems Manager (SSM) is a legitimate remote-administration channel, and it is one of the cleanest ways to run code on EC2 from the API alone. Any instance running the SSM agent with a role that allows it is reachable: no SSH key, no open port, no touching the metadata service. Command execution lands as root or SYSTEM, which then yields the instance role and on-host secrets.

## What folds in here

- **[Run command](run-command.md)**: `ssm:SendCommand` to execute commands on one or many managed instances.
- **[Session Manager](session-manager.md)**: `ssm:StartSession` to open an interactive shell.

## Finding managed instances

```bash
aws ssm describe-instance-information --query 'InstanceInformationList[].InstanceId'
```

## References

- [HackTricks Cloud: SSM abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [AWS: Systems Manager Run Command](https://docs.aws.amazon.com/systems-manager/latest/userguide/execute-remote-commands.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
