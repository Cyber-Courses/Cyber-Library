---
title: "Resource policy: invoking a private API through a permissive policy"
description: "Abusing a permissive API Gateway resource policy to invoke private or internal APIs."
keywords:
  - API Gateway
  - resource policy
  - private API
  - invoke
  - access
---

# Resource policy

An API Gateway **resource policy** decides which principals, source VPCs, or IP ranges may invoke the API. When it is written with a wildcard principal, a missing condition, or an overly broad source, an API meant to be private is invokable by anyone who can reach its endpoint, including internal-only APIs exposed to an attacker who holds any account credential or network position.

## Reading the policy and invoking

```bash
aws apigateway get-rest-api --rest-api-id <id> --query 'policy'
# invoke the stage endpoint directly
curl -s "https://<id>.execute-api.<region>.amazonaws.com/<stage>/<path>"
```

For a `PRIVATE` API tied to a VPC endpoint, invoke it from inside the VPC (or through SSRF from a resource there) when the policy does not further restrict the caller.

## Exploitation notes

- A `Principal: "*"` with no `Condition` makes the API world-invokable at its endpoint; a `aws:SourceVpce` condition only helps the attacker who is already inside that VPC.
- Private APIs are reached from a compromised in-VPC resource or an SSRF primitive, so pair this with [SSRF](../../credentials/instance-metadata/ssrf.md).
- Map routes first with `get-resources`; undocumented routes behind the gateway are frequently the interesting ones.

## Tools

- **AWS CLI** (`apigateway get-rest-api`, `get-resources`): policy and route discovery.
- **curl** / **Burp Suite**: invoking and fuzzing the exposed routes.

## References

- [AWS: API Gateway resource policy examples](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-resource-policies-examples.html)
- [HackTricks Cloud: AWS API Gateway](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
