---
title: "Management plane: Prism Element and Prism Central"
description: "Abusing the Nutanix management plane: Prism Element on each cluster and Prism Central across many clusters, and their REST API, to control VMs, hosts, and storage with recovered or weak credentials, where admin access reaches the whole estate."
keywords:
  - Prism
  - Prism Central
  - Nutanix API
  - management plane
  - HCI
---

# Management plane

Nutanix is managed through Prism: Prism Element runs per cluster, and Prism Central manages many clusters from one place. Both expose a web UI and a REST API that control VMs, hosts, storage, and users. Admin access to Prism, especially Prism Central, is control of the entire virtual estate, and the API also reaches guest operations.

```bash
# Prism REST API with recovered credentials
curl -sk -u 'admin:pass' -H 'Content-Type: application/json' https://<prism>:9440/api/nutanix/v3/vms/list -X POST -d '{"kind":"vm"}'
curl -sk -u 'admin:pass' https://<prism>:9440/PrismGateway/services/rest/v2.0/vms/
```

## Exploitation notes

- Prism Central is the top target: it spans clusters, so its admin reaches far more than a single Prism Element.
- The REST API performs VM lifecycle, snapshot, and storage operations, a management path to guest data without an escape.
- Default or weakly rotated Prism credentials, and tokens in automation, are common entry points.

## References

- [Nutanix REST API](https://www.nutanix.dev/api-reference/)
- [Nutanix: Prism administration](https://portal.nutanix.com/)
