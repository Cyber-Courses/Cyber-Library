---
title: "API Gateway"
order: 3
description: "Attacking API Gateway: resource-policy exposure, authorizer bypass, and the integration role that backend calls run under."
keywords:
  - API Gateway
  - resource policy
  - authorizer
  - integration role
  - REST API
---

# API Gateway

API Gateway fronts Lambda functions and other AWS services behind an HTTP API, and three things on it are attacker-relevant: the **resource policy** that decides who may invoke it, the **authorizer** that is supposed to gate protected routes, and the **integration role** that backend calls execute under. A loose resource policy exposes a private API, a flawed authorizer is bypassed to reach protected routes, and a powerful integration role turns a reachable endpoint into actions against other services.

## What folds in here

- **[Resource policy](resource-policy.md)**: a permissive policy that exposes a private or internal API.
- **[Authorizer bypass](authorizer-bypass.md)**: defeating a Lambda or JWT authorizer to reach protected routes.
- **[Integration role](integration-role.md)**: the credentials role that proxied AWS-service calls run under.

## Enumerating APIs

```bash
aws apigateway get-rest-apis
aws apigatewayv2 get-apis
aws apigateway get-resources --rest-api-id <id>
aws apigateway get-method --rest-api-id <id> --resource-id <rid> --http-method GET
# steal usable API keys, values included
aws apigateway get-api-keys --include-values
```

A key returned with its value is used directly as the `x-api-key` header against the stage, so `get-api-keys --include-values` turns read access into working calls.

## References

- [HackTricks Cloud: AWS API Gateway](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: API Gateway resource policies](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-resource-policies.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [fwd:cloudsec talks](https://fwdcloudsec.org/)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
