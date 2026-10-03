---
title: "Global Accelerator: fronting endpoints through listeners and endpoint groups"
description: "Abusing Global Accelerator listeners and endpoint groups to reach or front internal endpoints."
keywords:
  - Global Accelerator
  - listener
  - endpoint group
  - anycast
  - routing
---

# Global Accelerator

Global Accelerator puts a pair of static anycast IPs in front of backend endpoints (ALBs, NLBs, EC2, Elastic IPs) and routes to them over the AWS backbone. With write access you add or repoint listeners and endpoint groups to front an endpoint you control, or to expose an internal backend through the accelerator's public addresses.

## Enumerating accelerators and backends

```bash
aws globalaccelerator list-accelerators
aws globalaccelerator list-listeners --accelerator-arn <arn>
aws globalaccelerator list-endpoint-groups --listener-arn <listener-arn> \
  --query "EndpointGroups[].EndpointDescriptions[].EndpointId"
```

## Exploitation notes

- Adding an endpoint to an existing group can place a backend you control behind a trusted, allow-listed accelerator IP.
- The static anycast IPs are frequently allow-listed by partners; fronting your endpoint behind them inherits that trust.
- Endpoint groups can reference backends in other regions, widening reach from a single accelerator.

## Tools

- **AWS CLI** (`globalaccelerator ...`): enumeration and listener/endpoint changes.
- **ScoutSuite** / **Prowler**: inventory accelerators and their endpoints.

## References

- [AWS: Global Accelerator components](https://docs.aws.amazon.com/global-accelerator/latest/dg/introduction-components.html)
- [HackTricks Cloud: AWS networking services](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-vpc-and-network-security.html)
