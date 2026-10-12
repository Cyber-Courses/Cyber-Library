---
title: "Divulgation d'informations : ce que révèle un parcours SNMP"
order: 5
description: "Une community string de lecture prend toute sa valeur grâce à ce que l'agent divulgue : le système et la topologie réseau, la configuration en cours complète d'un équipement (exfiltrée sur Cisco via SNMP et TFTP), des données d'hôte Windows étendues via les MIB host-resources et LanMgr, et des identifiants et secrets stockés dans des OID standard et personnalisés."
keywords:
  - snmp disclosure
  - topology
  - running config
  - credentials
  - host resources
---

# Divulgation d'informations

L'énumération est le mécanisme ; la divulgation est le gain. Ce qu'un agent SNMP renvoie à une chaîne de lecture est fréquemment sensible et parfois catastrophique. Les tables réseau reconstruisent la topologie interne, qui est connecté, le routage et les correspondances ARP, cartographiant l'environnement à partir d'un seul équipement. Sur le matériel réseau, la configuration en cours elle-même peut être récupérée via SNMP, exposant chaque identifiant et chaque clé qu'elle contient. Sur les hôtes Windows, des MIB étendues énumèrent les utilisateurs, les processus, les logiciels installés et les partages. Et des identifiants et secrets, d'autres community strings, des clés Wi-Fi et VPN ainsi que des données applicatives apparaissent dans des OID standard et fournisseur. Cette section traite de la transformation d'un accès en lecture en ce butin.

## Sous-sujets

- **[Divulgation du système et de la topologie](system-and-topology-disclosure.md)** : cartographier le réseau à partir des tables de l'équipement.
- **[Exfiltration de configuration Cisco](cisco-configuration-exfiltration.md)** : récupérer la configuration en cours via SNMP et TFTP.
- **[Informations sur l'hôte Windows](windows-host-information.md)** : utilisateurs, processus, logiciels et partages.
- **[Exposition d'identifiants et de secrets](credential-and-secret-exposure.md)** : clés et identifiants dans les OID.

## Références

- [RFC 1213 (MIB-II)](https://datatracker.ietf.org/doc/html/rfc1213)
- [HackTricks: SNMP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
