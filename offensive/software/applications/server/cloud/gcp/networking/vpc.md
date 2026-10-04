---
title: "VPC: pivoting through networks, peering, and shared VPC"
description: "Pivoting through VPC networks, peering, and shared VPC to reach internal instances and services."
keywords:
  - VPC
  - peering
  - shared VPC
  - internal
  - pivot
  - network
---

# VPC

A GCP VPC is a global network of regional subnets. From a foothold on one instance, the reachable set is defined by the VPC, its **peerings**, any **shared VPC** attachment, and its routes. Enumerating that topology turns a single compromised host into a map of internal services to pivot toward.

## Mapping the reachable network

```bash
gcloud compute networks list
gcloud compute networks subnets list --format="table(name,region,network,ipCidrRange)"

# peerings extend reach into other VPCs (often other projects)
gcloud compute networks peerings list --network <vpc>

# shared VPC: a host project lends subnets to service projects
gcloud compute shared-vpc get-host-project <project>
gcloud compute shared-vpc associated-projects list <host-project>
```

## Reaching internal hosts

```bash
# internal addresses of instances you can now route to
gcloud compute instances list \
  --format="table(name,networkInterfaces[].networkIP.list(),networkInterfaces[].network.list())"
```

With a foothold instance, use it as the pivot (SSH local/dynamic forward, or an implant) to reach internal-only services across peered and shared networks.

## Exploitation notes

- VPC **peering** is non-transitive but often chains in practice through a hub network; follow the peering graph to find the widest-reach instance.
- **Shared VPC** means a service-project instance sits on the host project's subnet, so compromising one service project can expose the shared internal range of others.
- Private Google Access and Cloud NAT give instances egress even without public IPs, which matters for your exfiltration and callback path.

## Tools

- **gcloud** (`compute networks`, `subnets`, `peerings`): topology enumeration.
- **SSH / proxychains**: pivot through a foothold instance into internal ranges.

## References

- [HackTricks Cloud: GCP VPC](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Google Cloud: VPC network peering](https://cloud.google.com/vpc/docs/vpc-peering)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
