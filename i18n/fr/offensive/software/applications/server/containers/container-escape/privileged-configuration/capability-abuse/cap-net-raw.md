---
title: "CAP_NET_RAW : sniffing et usurpation depuis l'intérieur du réseau du conteneur"
order: 7
description: "CAP_NET_RAW permet la création de sockets raw et packet. Elle fait partie de l'ensemble par défaut de Docker, donc elle est largement disponible : un attaquant l'utilise pour sniffer le trafic du réseau du conteneur, forger des réponses ARP et DNS pour rediriger les voisins, et fabriquer des paquets arbitraires, rendant possibles l'homme du milieu et le mouvement latéral à travers le réseau du conteneur ou du pod, même sans évasion vers l'hôte."
keywords:
  - cap_net_raw
  - socket raw
  - usurpation ARP
  - usurpation DNS
  - réseau de conteneur
---

# CAP_NET_RAW

`CAP_NET_RAW` autorise les sockets `AF_PACKET` et `AF_INET` raw, qui lisent et écrivent directement les trames des couches liaison et réseau. Contrairement aux autres capacités de ces pages, elle ne s'évade pas vers le noyau de l'hôte ; elle compromet plutôt le réseau du conteneur. Comme elle fait partie de l'ensemble de capacités par défaut de Docker, presque chaque conteneur en dispose, ce qui fait de l'interception réseau un premier mouvement fiable pour pivoter entre conteneurs sur un pont partagé ou entre pods sur un nœud partagé.

Confirmez la capacité et inspectez le segment local :

```bash
capsh --print | grep -o cap_net_raw
grep CapEff /proc/self/status          # bit 13 set
ip -4 neigh; ip -4 addr               # neighbours and local subnet
```

## Voie : usurpation ARP pour l'homme du milieu

Sur un pont de niveau 2 partagé (le réseau `bridge` par défaut de Docker, ou des pods sur un même nœud), l'attaquant forge des réponses ARP pour associer l'IP de la passerelle d'une victime à sa propre adresse MAC, de sorte que le trafic de la victime transite par le conteneur de l'attaquant.

```bash
# enable forwarding so the victim keeps connectivity while you intercept
sysctl -w net.ipv4.ip_forward=1 2>/dev/null
# poison the victim's view of the gateway, and the gateway's view of the victim
arpspoof -i eth0 -t 172.17.0.3 172.17.0.1 &
arpspoof -i eth0 -t 172.17.0.1 172.17.0.3 &
tcpdump -i eth0 -w loot.pcap host 172.17.0.3   # capture the redirected traffic
```

## Voie : usurpation DNS et fabrication de paquets

Les sockets raw permettent à l'attaquant de répondre aux requêtes DNS des voisins avant le vrai résolveur, pointant un nom de service vers une adresse contrôlée par l'attaquant afin de récolter des identifiants dans des protocoles en clair ou rétrogradés en TLS. La même primitive forge des paquets pour sonder ou exploiter des services qui font confiance aux adresses source sur le réseau interne.

```bash
dnsspoof -i eth0 -f hosts.txt       # answer selected names with attacker IPs
```

## Notes d'exploitation

- C'est du mouvement latéral au sein du réseau du conteneur ou du pod, pas une évasion vers l'hôte. Sa valeur est la récolte d'identifiants et de jetons en transit (HTTP interne, base de données, ou trafic de métadonnées) qui débloquent ensuite d'autres hôtes.
- Les réseaux de conteneurs sont fréquemment plats, de sorte qu'un seul conteneur par défaut compromis peut intercepter le trafic de tous ses voisins sur le même pont ; dans Kubernetes sans politique réseau, les pods d'un nœud partagent leur accessibilité.
- Retirer cette capacité (`--cap-drop=NET_RAW`) est une étape de durcissement courante, donc sa présence n'est pas garantie sur des charges de travail verrouillées ; confirmez-la avant de vous y fier.

## Outils

- [dsniff (arpspoof, dnsspoof)](https://www.monkey.org/~dugsong/dsniff/)
- [Scapy (fabrication de paquets raw)](https://scapy.net/)

## Références

- [man 7 raw](https://man7.org/linux/man-pages/man7/raw.7.html)
- [man 7 packet](https://man7.org/linux/man-pages/man7/packet.7.html)
- [HackTricks : CAP_NET_RAW](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities#cap_net_raw)
