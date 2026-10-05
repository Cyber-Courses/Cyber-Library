---
title: "SageMaker: stealing data, models, and the notebook role"
description: "Abusing SageMaker notebooks, training jobs, and endpoints to steal data, models, and the attached role."
keywords:
  - SageMaker
  - notebook
  - training job
  - model
  - role
---

# SageMaker

SageMaker runs notebooks, training jobs, and inference endpoints, each attached to an execution role with access to the training data in S3. Two things are worth taking: the **data and models**, and the **execution role**. A principal with `sagemaker:CreatePresignedNotebookInstanceUrl` opens a shell in a running notebook and reads its role credentials from the metadata service, which is the same `PassRole` escalation documented under [identity](../identity/privilege-escalation/pass-role/sagemaker.md).

## Opening a notebook you can reach

```bash
aws sagemaker list-notebook-instances
aws sagemaker create-presigned-notebook-instance-url --notebook-instance-name <nb>
# open the URL, drop to a terminal, then read the attached role from IMDS
```

## Reading data and models

```bash
aws sagemaker list-training-jobs
aws sagemaker describe-training-job --training-job-name <j> \
  --query 'InputDataConfig[].DataSource.S3DataSource.S3Uri'   # training data in S3
aws sagemaker list-models ; aws sagemaker describe-model --model-name <m>
```

## Exploitation notes

- The notebook's execution role is usually broad (full S3, sometimes more); stealing it from IMDS turns data access into account movement.
- Training-job and model definitions point straight at the S3 buckets holding the data, which you then read with the role you just took.
- Creating a new notebook with a passed privileged role is the louder `PassRole` variant when you cannot reach an existing one.

## Tools

- **AWS CLI** (`sagemaker create-presigned-notebook-instance-url`, `describe-training-job`).
- **Pacu** (`sagemaker__*`): enumerate notebooks, jobs, and models.

## References

- [HackTricks Cloud: AWS SageMaker](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: SageMaker notebook instance security](https://docs.aws.amazon.com/sagemaker/latest/dg/security.html)
- [CloudFox: enumerating roles and data behind compute services](https://github.com/BishopFox/cloudfox)
- [Datadog Security Labs: AWS attack research](https://securitylabs.datadoghq.com/)
