---
title: "SageMaker: PassRole into a notebook or training job"
description: "iam:PassRole into a SageMaker notebook instance or training job to execute code under a privileged SageMaker role."
keywords:
  - PassRole
  - SageMaker
  - notebook
  - training job
  - service role
---

# SageMaker

SageMaker notebook instances and training jobs run under a passed **execution role**. With `iam:PassRole` and `sagemaker:CreateNotebookInstance` (plus `CreatePresignedNotebookInstanceUrl`), you attach a privileged role to a notebook you control and open a Jupyter terminal that holds the role's credentials.

## Notebook path

```bash
aws sagemaker create-notebook-instance --notebook-instance-name x \
  --instance-type ml.t3.medium \
  --role-arn arn:aws:iam::<acct>:role/<privileged-role>
aws sagemaker create-presigned-notebook-instance-url --notebook-instance-name x
# open the URL, launch a terminal, then:
#   aws sts get-caller-identity   (runs as the SageMaker role)
```

## Training-job path

`sagemaker:CreateTrainingJob` runs a container you specify as the role, with no interactive access needed; have the container entrypoint exfiltrate the role credentials.

## Exploitation notes

- The presigned URL grants access without SageMaker console permissions, so one API call plus the URL is the whole path.
- SageMaker roles commonly carry broad S3 and ECR access for datasets and images, extending the pivot.

## Tools

- **AWS CLI** (`sagemaker create-notebook-instance` / `create-presigned-notebook-instance-url`).
- **Pacu**: SageMaker privesc module.

## References

- [Rhino Security Labs: AWS privilege escalation (SageMaker)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
