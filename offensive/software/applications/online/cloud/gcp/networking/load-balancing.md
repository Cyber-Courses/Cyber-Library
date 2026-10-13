---
title: "Load balancing: reaching backends through forwarding rules"
order: 4
description: "Mapping and reaching backends through GCP load balancers, exposed forwarding rules, and backend services."
keywords:
  - load balancing
  - forwarding rule
  - backend service
  - exposure
  - GCP
  - ingress
---

# Load balancing

A GCP load balancer is assembled from forwarding rules, target proxies, URL maps, and backend services that point at instance groups, network endpoint groups, or buckets. Enumerating these reveals the public frontends and the backends they reach, and misconfigured backends or origin-direct access let you bypass the fronting controls.

## Mapping frontends to backends

```bash
# public entry points
gcloud compute forwarding-rules list \
  --format="table(name,IPAddress,IPProtocol,portRange,target)"

# what each backend service points at, and its security policy
gcloud compute backend-services list \
  --format="table(name,protocol,backends[].group.list(),securityPolicy)"
gcloud compute url-maps list
```

## Exploitation notes

- A backend service with no attached Cloud Armor **security policy** is reachable without WAF filtering; a policy attached only at the load balancer is bypassed by reaching the backend instance's own IP directly (pair with [firewall rules](firewall-rules.md)).
- Internal load balancers expose services only inside the VPC, so they become reachable after a [VPC](vpc.md) pivot, not from the internet.
- A backend bucket (`gcloud compute backend-buckets list`) fronts a GCS bucket through the load balancer; the underlying bucket may also be reachable directly, see [Cloud Storage](../storage/cloud-storage/index.md).

## Tools

- **gcloud** (`compute forwarding-rules`, `backend-services`, `url-maps`): frontend and backend enumeration.
- **nmap** / **curl**: probe the forwarding-rule IPs and test origin-direct reachability.

## References

- [HackTricks Cloud: GCP load balancing](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Google Cloud: cloud load balancing overview](https://cloud.google.com/load-balancing/docs/load-balancing-overview)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
