---
title: "Authorizer bypass: defeating a Lambda or JWT authorizer"
order: 2
description: "Bypassing a Lambda or JWT authorizer through caching, header handling, or logic flaws to reach protected routes."
keywords:
  - authorizer
  - JWT
  - Lambda authorizer
  - bypass
  - API Gateway
---

# Authorizer bypass

A custom **authorizer** (a Lambda that returns an IAM policy, or a JWT authorizer) is supposed to gate protected routes. Three recurring weaknesses defeat it: authorization-result **caching** keyed on an identity source an attacker controls, loose **token or header handling** in a Lambda authorizer, and **policy-scope** flaws where the returned policy allows more than the requested method.

## Probing the authorizer

```bash
aws apigateway get-authorizers --rest-api-id <id>
# how the identity source is keyed drives the cache attack
aws apigateway get-authorizer --rest-api-id <id> --authorizer-id <aid> \
  --query '[identitySource,authorizerResultTtlInSeconds,type]'
```

## Common bypasses

```
# cache poisoning: if the result is cached per token value, a once-valid
# token is replayed within the TTL even after the session should be dead
Authorization: <previously-valid-token>

# Lambda authorizer that greps for a substring or trusts an unverified header
X-Forwarded-For, X-Original-Url, or a crafted token the handler parses loosely

# wildcarded returned policy: methodArn "*/*" lets one authorized route reach all
```

## Exploitation notes

- When `authorizerResultTtlInSeconds` is high and the identity source is the token itself, a leaked or replayed token stays good for the whole TTL.
- Lambda authorizers that build the allow policy with a wildcard `Resource` grant every route once any route authorizes.
- JWT authorizers that do not pin `aud`/`iss`, or accept `alg: none`, are broken the same way as any JWT verifier; see the web JWT pages.

## Tools

- **AWS CLI** (`apigateway get-authorizers`): read the authorizer config.
- **Burp Suite** / **jwt_tool**: craft and replay tokens and headers.

## References

- [AWS: API Gateway Lambda authorizers](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-use-lambda-authorizer.html)
- [HackTricks Cloud: AWS API Gateway](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [fwd:cloudsec talks](https://fwdcloudsec.org/)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
