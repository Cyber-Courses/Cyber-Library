---
title: "Protocoles : attaquer les protocoles de télémétrie de supervision"
order: 1
description: "Les protocoles réseau qui transportent la télémétrie de supervision sont des surfaces d'attaque en eux-mêmes : SNMP expose les données et la configuration des équipements et est souvent faiblement authentifié, l'ingestion syslog fait confiance à son entrée et est usurpable et injectable, et les collecteurs NetFlow/IPFIX détiennent la carte du trafic du réseau. Chacun divulgue des informations ou permet à un attaquant d'empoisonner ce que voient les opérateurs."
keywords:
  - monitoring protocols
  - snmp
  - syslog
  - netflow
  - telemetry
---

# Protocoles

Avant les plateformes, les protocoles de télémétrie eux-mêmes sont attaquables. SNMP est le plus riche : un protocole largement déployé, fréquemment faiblement authentifié, qui révèle l'état complet d'un équipement et, avec un accès en écriture, le reconfigure. Syslog est une voie d'ingestion confiante, généralement non authentifiée, qu'un attaquant usurpe, falsifie et inonde pour empoisonner l'enregistrement sur lequel s'appuient les opérateurs et les SIEM. Et les collecteurs NetFlow/IPFIX détiennent la carte de qui parle à qui à travers le réseau, une reconnaissance précieuse et une cible pour les enregistrements falsifiés. Ce sont des attaques au niveau du protocole, indépendantes de la plateforme qui les consomme.

## Sous-thèmes

- **[SNMP](snmp/index.md)** : divulgation des données d'équipement, chaînes de communauté faibles et abus de l'accès en écriture.
- **[Syslog](syslog/index.md)** : usurpation, injection et inondation du flux de journaux.
- **[NetFlow et IPFIX](netflow-and-ipfix/index.md)** : récolte et falsification de la télémétrie des flux réseau.

## Références

- [SNMP (RFC 1157, RFC 3416)](https://datatracker.ietf.org/doc/html/rfc3416)
- [Syslog (RFC 5424)](https://datatracker.ietf.org/doc/html/rfc5424)
- [IPFIX (RFC 7011)](https://datatracker.ietf.org/doc/html/rfc7011)
