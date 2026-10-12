---
title: "Capture de bannière : divulgation du service et de la version Telnet"
description: "Un serveur Telnet envoie généralement une bannière à la connexion, qui nomme souvent le système d'exploitation, le modèle de l'équipement et la version du logiciel, les équipements réseau et les appareils IoT étant particulièrement bavards. Cette bannière identifie la cible pour la recherche d'identifiants par défaut et pour faire correspondre les anciennes versions de telnetd à leurs vulnérabilités de corruption mémoire."
keywords:
  - telnet banner
  - divulgation de version
  - identification d'équipement
  - nc 23
  - fingerprint
---

# Capture de bannière

La connexion à Telnet produit presque toujours une bannière avant ou au niveau de l'invite de connexion, et elle est fréquemment riche : elle nomme le système d'exploitation (une bannière de connexion Linux, un en-tête Cisco IOS, une invite BusyBox), le fournisseur et le modèle de l'équipement pour le matériel réseau et l'IoT, et parfois la version exacte du firmware ou du logiciel. C'est l'empreinte principale pour Telnet, car l'équipement et la version correspondent directement aux identifiants par défaut documentés et à l'implémentation `telnetd` spécifique (et à ses bugs RCE connus).

```bash
nc <target> 23                                 # read the banner/login header
echo | nc -w3 <target> 23 | head
nmap -p23 -sV <target>                         # version detection
# at scale
for h in $(cat hosts.txt); do echo "== $h =="; echo | nc -w2 $h 23 | head -5; done
```

## Notes d'exploitation

- La bannière identifie couramment la classe d'équipement (routeur/switch/caméra/imprimante/IoT) et le fournisseur, ce qui est exactement ce dont une recherche d'[identifiants par défaut](../authentication/default-credentials.md) a besoin.
- Les chaînes de firmware/version associent un `telnetd` embarqué (souvent BusyBox ou une version du fournisseur) à ses failles de corruption mémoire connues, voir [Corruption mémoire](../memory-corruption/index.md).
- Les bannières Cisco et autres équipements réseau révèlent la famille du système d'exploitation et parfois le modèle, orientant à la fois les choix d'identifiants et d'exploitation.
- Tout ceci est pré-authentification et en clair ; cela ne coûte rien et devrait précéder toute tentative d'authentification.

## Références

- [HackTricks: Telnet banner](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [RFC 854 (Telnet)](https://datatracker.ietf.org/doc/html/rfc854)
