---
title: "NetFlow et IPFIX : attaquer la télémétrie des flux réseau"
order: 3
description: "Les protocoles d'export de flux (NetFlow, sFlow, IPFIX) résument les conversations réseau et les envoient à un collecteur. Ces données sont une carte de qui parle à qui à travers l'environnement, une reconnaissance précieuse lorsqu'elle est récoltée depuis un collecteur exposé, et la voie d'export non authentifiée permet à un attaquant de falsifier des enregistrements de flux pour injecter du faux trafic ou tromper la supervision."
keywords:
  - netflow
  - ipfix
  - sflow
  - flow collector
  - telemetry
---

# NetFlow et IPFIX

Les protocoles d'export de flux, Cisco NetFlow, sFlow et l'IPFIX standardisé par l'IETF, font résumer par les routeurs et les commutateurs les conversations qui les traversent (source, destination, ports, protocole, nombre d'octets et de paquets) et exportent ces enregistrements vers un collecteur en UDP. Pour un attaquant, les données de flux constituent une carte toute faite des communications du réseau : qui parle à qui, sur quels services, et dans quelle mesure, ce qui est une reconnaissance de grande valeur obtenue sans toucher aux hôtes. Récolter cette carte depuis un collecteur exposé ou son stockage révèle la structure et les relations de l'environnement. Et comme la voie d'export est de l'UDP non authentifié, un attaquant capable d'émettre vers le collecteur falsifie des enregistrements de flux pour injecter du faux trafic, masquer des flux réels, ou tromper la supervision de capacité et de sécurité construite sur ces données.

## Sous-thèmes

- **[Récolte des données du collecteur](collector-data-harvesting.md)** : lire la carte du trafic depuis un collecteur.
- **[Falsification d'enregistrements de flux](flow-record-spoofing.md)** : falsifier les enregistrements de flux exportés.

## Références

- [RFC 7011 (IPFIX)](https://datatracker.ietf.org/doc/html/rfc7011)
- [Cisco NetFlow overview](https://www.cisco.com/c/en/us/products/ios-nx-os-software/ios-netflow/index.html)
