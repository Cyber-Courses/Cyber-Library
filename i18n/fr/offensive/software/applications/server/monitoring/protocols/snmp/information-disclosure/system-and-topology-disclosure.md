---
title: "Divulgation du système et de la topologie : cartographier le réseau via SNMP"
order: 1
description: "Les tables standard de MIB-II reconstruisent le réseau interne à partir d'un seul équipement : les interfaces et leurs adresses, la table de routage et la table ARP (IP vers MAC). Les parcourir sur un routeur ou un commutateur cartographie les sous-réseaux adjacents, les passerelles et les hôtes actifs, offrant à un attaquant la topologie interne sans balayage, à partir d'un accès SNMP en lecture seule."
keywords:
  - topology
  - iftable
  - iproutetable
  - arp table
  - network map
---

# Divulgation du système et de la topologie

Un seul équipement réseau atteignable, un routeur, un commutateur ou un pare-feu, divulgue une grande partie du réseau environnant à travers les tables standard de MIB-II, de sorte qu'une community string de lecture devient une carte de l'environnement sans aucun balayage actif. La table des interfaces (`ifTable`, `1.3.6.1.2.1.2`) et la table des adresses IP (`ipAddrTable`, `1.3.6.1.2.1.4.20`) donnent les interfaces de l'équipement et les sous-réseaux sur lesquels il se trouve. La table de routage (`ipRouteTable`/`ipCidrRouteTable`) révèle les passerelles et les réseaux atteignables, esquissant la topologie au-delà de l'équipement. Et la table ARP/voisins (`ipNetToMediaTable`, `1.3.6.1.2.1.4.22`) liste les correspondances IP vers MAC, ce qui constitue un inventaire tout prêt des hôtes actifs sur les segments de l'équipement.

```bash
# interfaces and the addresses/subnets the device is on
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.2.2.1.2     # ifDescr (interface names)
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.4.20.1.1    # ipAdEntAddr (IP addresses)
# routing table: gateways and reachable networks
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.4.21.1      # ipRouteTable
# ARP table: live IP->MAC neighbours (a host inventory)
snmpbulkwalk -v2c -c public <target> 1.3.6.1.2.1.4.22.1.3    # ipNetToMediaNetAddress
# snmp-check formats much of this automatically
snmp-check -c public <target>
```

## Notes d'exploitation

- Un routeur ou un commutateur de niveau 3 est la meilleure cible : sa table de routage esquisse toute la topologie interne et sa table ARP énumère les hôtes actifs sur chaque segment connecté, soit de la reconnaissance que l'on n'obtiendrait autrement qu'en balayant.
- La table ARP/voisins est de fait une liste de découverte passive des hôtes pour les sous-réseaux de l'équipement ; recoupez-la avec la table de routage pour prioriser les points de pivot.
- C'est une opération en lecture seule et discrète (aucun balayage des hôtes eux-mêmes), elle cartographie donc le réseau avant que vous n'y touchiez.
- À combiner avec le butin plus riche propre à chaque hôte : [Informations sur l'hôte Windows](windows-host-information.md) sur les serveurs et [Exfiltration de configuration Cisco](cisco-configuration-exfiltration.md) sur le matériel Cisco.

## Références

- [RFC 1213 (MIB-II IP group)](https://datatracker.ietf.org/doc/html/rfc1213)
- [HackTricks: SNMP topology](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
