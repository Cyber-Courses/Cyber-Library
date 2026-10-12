---
title: "Réseau : techniques offensives par support de transport"
description: "La catégorie Réseau couvre le travail offensif contre les protocoles et l'infrastructure réseau, répartie par support entre filaire et sans fil, car l'accès et l'outillage diffèrent fondamentalement de l'un à l'autre."
keywords:
  - attaques réseau
  - réseau filaire
  - attaques sans fil
  - abus de protocole
  - pivotement réseau
---

# Réseau

La catégorie Réseau couvre le travail offensif contre les liens, protocoles et infrastructures qui déplacent les données entre les systèmes. Elle recouvre la découverte et la cartographie, l'interception et la manipulation du trafic, l'abus du comportement des protocoles, et l'usage d'un point d'ancrage sur un segment pour en atteindre un autre. La cible ici est le réseau lui-même, et non les hôtes qui s'y trouvent, lesquels sont traités sous [logiciel](../software/index.md).

## Pourquoi ce découpage

Les deux sous-catégories se divisent selon le support de transport, car la façon dont un attaquant accède au trafic, et l'équipement requis, diffèrent fondamentalement :

- **Filaire** : des attaques qui nécessitent un point d'ancrage physique ou logique sur un segment filaire, y compris la manipulation de la couche 2 (ARP, VLAN, spanning tree), les dispositifs malveillants et l'interception sur réseaux commutés.
- **Sans fil** : des attaques qui exploitent la radio comme support partagé, y compris les attaques sur l'association et les clés Wi-Fi, les points d'accès malveillants et l'abus d'autres protocoles radio, où la proximité remplace le câble.

Le découpage importe parce que le premier problème d'une attaque réseau est d'atteindre le trafic tout court. Sur un réseau filaire cela signifie un port, une prise d'écoute ou une position d'homme du milieu ; en sans fil cela signifie être à portée et déjouer les protections de la couche liaison. Une fois le trafic atteint, les techniques d'abus de protocole convergent souvent, mais la phase d'accès qui les précède est propre au support, ce qui explique que la catégorie se ramifie d'abord à cet endroit.

## Références

- [OWASP: Testing Network Infrastructure](https://owasp.org/www-project-web-security-testing-guide/)
- [MITRE ATT&CK: Network-based techniques](https://attack.mitre.org/)
