---
title: "GCP networking"
order: 7
description: "Attacking GCP networking: firewall-rule exposure, VPC reach, Cloud DNS and subdomain takeover, and load-balancer exposure."
keywords:
  - firewall rules
  - VPC
  - Cloud DNS
  - load balancing
  - subdomain takeover
  - networking
---

# Networking

GCP networking decides what is reachable: **firewall rules** that open ingress to the internet, **VPC** peering and shared VPC that let a foothold in one network reach another, **Cloud DNS** records left dangling at released resources, and **load balancers** that front backends. Networking rarely compromises a project on its own, but it turns a reachable foothold into reach across the environment and captures trusted names.

## What folds in here

- **[Firewall rules](firewall-rules.md)**: opening ingress with `compute.firewalls.create`/`update`, and finding rules already open to `0.0.0.0/0`.
- **[VPC](vpc.md)**: peering, shared VPC, and routes to pivot to internal instances and services.
- **[Cloud DNS](cloud-dns.md)**: hijacking dangling records that point at released GCP resources.
- **[Load balancing](load-balancing.md)**: forwarding rules and backend services that expose or reach backends.

Identity-based movement (impersonation and `actAs` across projects) lives in [identity](../identity/index.md); this surface is the network layer.

## References

- [HackTricks Cloud: GCP networking](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google Cloud: VPC firewall rules](https://cloud.google.com/firewall/docs/firewalls)
