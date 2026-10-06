---
title: "VPC: pivoting through peering, endpoints, and routing"
order: 2
description: "Pivoting through VPC peering, endpoints, and routing to reach internal services and other accounts."
keywords:
  - VPC
  - peering
  - endpoint
  - routing
  - pivot
---

# VPC

Once you hold an instance or a role inside a VPC, the network layout decides how far you can reach. Peering connections, transit gateways, and routing tables stitch VPCs together (often across accounts), and VPC endpoints expose AWS service access paths. Reading the routing and peering graph with `ec2:Describe*` shows which internal ranges and accounts are reachable from your foothold.

## Reading the reachability graph

```bash
aws ec2 describe-vpc-peering-connections \
  --query "VpcPeeringConnections[].[VpcPeeringConnectionId,RequesterVpcInfo.CidrBlock,AccepterVpcInfo.CidrBlock]"
aws ec2 describe-route-tables \
  --query "RouteTables[].Routes[].[DestinationCidrBlock,GatewayId,VpcPeeringConnectionId,TransitGatewayId]"
aws ec2 describe-transit-gateway-attachments
```

## Endpoints and interface services

```bash
# interface/gateway endpoints reveal which services the VPC can reach privately
aws ec2 describe-vpc-endpoints \
  --query "VpcEndpoints[].[ServiceName,VpcEndpointType,VpcId]"
```

## Exploitation notes

- A peering route to another account's CIDR is a cross-account pivot: scan that range from your foothold once a route exists.
- Transit gateways concentrate reachability; one attachment can expose many VPCs at once.
- Interface endpoints with permissive endpoint policies let an in-VPC principal reach services the account meant to keep private.

## Tools

- **AWS CLI** (`ec2 describe-*`): the routing and peering graph.
- **ScoutSuite**: visualize peering and endpoint exposure.
- **pmapper**: cross-account IAM edges that often parallel the network peering.

## References

- [HackTricks Cloud: AWS VPC and networking](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-vpc-and-network-security.html)
- [AWS: VPC peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)
- [CloudFox: mapping reachable cloud network paths](https://github.com/BishopFox/cloudfox)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
