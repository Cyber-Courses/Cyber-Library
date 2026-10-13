---
title: "Nagios et Icinga : attaquer les serveurs de supervision"
order: 2
description: "Nagios (Core et la version commerciale XI) et le fork Icinga supervisent les hôtes à travers des plugins et l'exécuteur distant NRPE. La surface, c'est l'authentification de l'interface web, l'agent NRPE sur le port 5666 qui exécute les plugins de vérification et est sujet à l'injection d'arguments et de commandes, l'injection dans les définitions de commandes et de macros de Nagios, et les chaînes récurrentes de Nagios XI qui mènent du contournement d'authentification à la RCE."
keywords:
  - nagios
  - nagios xi
  - icinga
  - nrpe
  - monitoring
---

# Nagios et Icinga

Nagios est un système de supervision de longue date qui existe sous deux formes : le logiciel libre Nagios Core (configuré par fichiers, avec une interface web CGI) et le produit commercial Nagios XI (une application web PHP complète par-dessus). Icinga est un fork avec Icinga 2 et l'interface Icinga Web 2. Ils supervisent les hôtes en exécutant des plugins de vérification localement et, pour les hôtes distants, via NRPE (le Nagios Remote Plugin Executor) sur le port TCP 5666. La surface d'attaque : l'authentification de l'interface web, l'agent NRPE qui exécute les commandes de vérification et constitue une cible classique d'injection de commandes, l'injection dans les définitions de commandes et de macros de Nagios qui s'exécutent sur le serveur de supervision, et la longue liste de vulnérabilités applicatives web de Nagios XI qui enchaînent contournement d'authentification et exécution de code à distance.

```bash
# web interfaces and the NRPE agent
curl -sk https://<target>/nagios/ ; curl -sk https://<target>/nagiosxi/
nmap -p5666 -sV <target>                         # NRPE
```

## Sous-thèmes

- **[Énumération](enumeration.md)** : identification du produit et de la version.
- **[Authentification](authentication.md)** : identifiants web par défaut et faibles.
- **[Abus de NRPE](nrpe-abuse.md)** : l'exécuteur de plugins distant sur le port 5666.
- **[Injection de commandes et de plugins](command-and-plugin-injection.md)** : injection dans les définitions de commandes.
- **[Exploits Nagios XI](nagios-xi-exploits.md)** : les chaînes du contournement d'authentification à la RCE.

## Références

- [Documentation Nagios](https://www.nagios.org/documentation/)
- [Documentation NRPE](https://github.com/NagiosEnterprises/nrpe)
- [HackTricks : 5666 NRPE](https://book.hacktricks.xyz/network-services-pentesting/5666-pentesting-nrpe)
