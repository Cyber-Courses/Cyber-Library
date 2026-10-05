---
title: "CNI and overlay abuse: spoofing and sniffing on the pod network"
description: "Abusing the Kubernetes CNI plugin and overlay network to spoof pod identities, sniff cross-node traffic on the overlay, and reach the node from the pod network, exploiting that many overlays provide confidentiality and identity only by convention."
keywords:
  - CNI
  - overlay network
  - pod spoofing
  - traffic sniffing
  - kubernetes network
---

# CNI and overlay abuse

The CNI plugin wires pods into the cluster network, usually through an overlay (VXLAN, IP-in-IP) or direct routing. Overlays typically carry traffic unencrypted and identify pods by IP, so a pod with enough network capability can sniff traffic traversing its node, spoof another pod's source address, or reach the node and the underlay.

```bash
# With CAP_NET_RAW or a host-network pod: observe overlay traffic on the node
tcpdump -i any -n 'udp port 4789' 2>/dev/null | head        # VXLAN
ip route; ip neigh                                           # overlay and node routes
```

## Exploitation notes

- Unencrypted overlays expose cross-pod traffic to anyone who can capture on the node, so pair this with a node foothold or a [Host network namespace](../../../container-escape/shared-host-namespaces/host-network-namespace.md) pod.
- IP-based identity lets a spoofed source impersonate a trusted pod to services that authorize by network origin.
- The underlay and node are reachable from the overlay on many setups, widening a pod foothold toward the node.

## References

- [Kubernetes: cluster networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
- [CNI specification](https://github.com/containernetworking/cni/blob/main/SPEC.md)
