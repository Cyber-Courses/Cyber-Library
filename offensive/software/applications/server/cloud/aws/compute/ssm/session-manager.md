---
title: "Session Manager: ssm:StartSession interactive shells"
description: "ssm:StartSession to open an interactive shell on a managed instance under the SSM agent's privileges."
keywords:
  - Session Manager
  - StartSession
  - shell
  - SSM agent
  - interactive
---

# Session Manager

`ssm:StartSession` opens an interactive shell on a managed instance, tunneled through the SSM service rather than a network port. It is the interactive counterpart to run command: the same agent privileges (root or SYSTEM), the same no-inbound-access property, but a live terminal for hands-on work.

## Opening a session

```bash
# needs the session-manager-plugin installed locally
aws ssm start-session --target <instance-id>

# port forwarding to reach an internal service through the instance
aws ssm start-session --target <id> \
  --document-name AWS-StartPortForwardingSession \
  --parameters '{"portNumber":["3306"],"localPortNumber":["3306"]}'
```

## Exploitation notes

- The port-forwarding document turns a managed instance into a pivot into private subnets without opening a security group.
- Sessions are logged to CloudTrail and optionally to S3 or CloudWatch; the run-command path can be quieter for a single action.
- As with run command, the shell runs as the agent and reaches the instance role through IMDS.

## Tools

- **AWS CLI** + **session-manager-plugin**: start sessions and port forwards.

## References

- [AWS: Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)
- [HackTricks Cloud: Session Manager](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
