---
title: "AWS compute"
order: 3
description: "Attacking AWS compute: EC2 user data and instance profiles, SSM run command and Session Manager, ECS and EKS containers and ECR images, and the Elastic Beanstalk, Image Builder, App Runner, Batch, and Lightsail runtimes."
keywords:
  - EC2
  - SSM
  - ECR
  - containers
  - instance profile
---

# Compute

Compute in AWS is where IAM meets code execution. Almost every compute service runs with an attached **role**, so taking over a workload, or launching one you control, yields that role's credentials and whatever it can reach. The same services also hold secrets at rest: user-data scripts, snapshots, images, and environment configuration. The pages here work both angles, stealing the role and reading the data.

## Surfaces

- **[EC2](ec2/index.md)**: user-data secrets and code, the instance profile role, and data recovery from EBS snapshots and shared AMIs.
- **[SSM](ssm/index.md)**: Systems Manager run command and Session Manager shells on managed instances, under the instance role.
- **[Containers](containers.md)**: ECS and EKS task-role theft, task-definition abuse, and the container runtime.
- **[ECR](ecr/index.md)**: pulling from exposed repositories and poisoning images that downstream compute will run.
- **[Elastic Beanstalk](elastic-beanstalk.md)**: environments, their instance-profile role, and secrets in configuration.
- **[Image Builder](image-builder.md)**: pipelines and components that bake code into golden AMIs.
- **[App Runner](app-runner.md)**: services and their instance role, and source or image deployment.
- **[Batch](batch.md)**: job definitions and compute environments running containers under a job role.
- **[Lightsail](lightsail.md)**: instances, keys, and snapshots that sit outside the main VPC and IAM visibility.

## References

- [HackTricks Cloud: AWS EC2, SSM and compute](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [Rhino Security Labs: AWS privilege escalation methods](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
