---
title: "Management plane: the Prism API and Controller VM services"
description: "Nutanix is managed through Prism Element (per cluster) and Prism Central (multi-cluster), REST APIs and web interfaces served by the Controller VMs. Access to Prism controls every VM and the storage fabric: creating and reconfiguring VMs, attaching and cloning vdisks, and reading cluster secrets, so Prism credentials or an API flaw is control of the estate."
keywords:
  - prism
  - prism central
  - nutanix api
  - v3 api
  - management plane
---

# Management plane

Nutanix management is Prism: Prism Element runs on each cluster's Controller VMs for local management, and Prism Central aggregates many clusters. Both expose REST APIs (the v1/v2 and v3 interfaces) and web UIs. Reaching Prism with credentials or through an API flaw is control of the virtualization estate: enumerate and manipulate every VM, create or reconfigure VMs, attach and clone the vdisks that hold guest data, manage images, and read cluster configuration and secrets. Prism Central is especially high-value because it spans clusters.

## Reach and drive Prism

```bash
# Prism REST API (served by the CVM / Prism Central), typically on 9440
curl -sk -u '<user>:<pass>' https://<prism>:9440/api/nutanix/v2.0/cluster/
curl -sk -u '<user>:<pass>' https://<prism>:9440/api/nutanix/v3/vms/list -XPOST -d '{"kind":"vm"}'
# enumerate VMs, images, and storage; then act
curl -sk -u '<u>:<p>' https://<prism>:9440/api/nutanix/v3/images/list -XPOST -d '{"kind":"image"}'
```

## Control actions

```text
With Prism access an attacker can:
- create or reconfigure a VM to attach/clone another VM's vdisk and read it
- clone or snapshot vdisks and export them (data theft without entering a VM)
- change VM boot/network settings, or deploy an attacker VM on the cluster
- read cluster configuration, users, and integration credentials
Prism Central extends all of this across every registered cluster.
```

## Exploitation notes

- Prism credentials are estate control; the v3 API in particular exposes VM, image, and storage management, so attaching or cloning a target VM's vdisk reads its data without a guest escape.
- Prism Central spans clusters, so compromising it multiplies reach; treat it as the highest-value management target.
- The APIs are served by the Controller VMs, so CVM access (from [Host access and shell](host-access-and-shell.md)) and Prism access reinforce each other, and local service credentials on a CVM can authenticate to Prism.
- Nutanix publishes security advisories for Prism and the APIs; fingerprint the version and match for any management-plane vulnerability.

## References

- [Nutanix v3 API reference](https://www.nutanix.dev/api-reference/)
- [Prism documentation](https://portal.nutanix.com/)
- [Nutanix security advisories](https://www.nutanix.com/support-services/security-advisories)
