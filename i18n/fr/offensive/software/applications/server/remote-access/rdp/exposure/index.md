---
title: "Exposition : RDP exposé sur internet et transport faible"
description: "RDP est couramment exposé directement sur internet sur le port 3389, où il est en permanence scanné, attaqué par force brute et par credential stuffing, et il constitue un vecteur d'entrée majeur pour les rançongiciels. L'analyse de l'exposition trouve ces points d'accès et évalue le transport : la sécurité RDP native et un TLS faible permettent l'attaque de la machine du milieu et le déchiffrement, aggravant les risques liés aux identifiants et à la pré-authentification."
keywords:
  - rdp exposure
  - internet-facing
  - port 3389
  - weak tls
  - attack surface
---

# Exposition

Le plus grand risque réel de RDP est une simple exposition : les organisations publient le port 3389 sur internet par commodité, et ces points d'accès sont perpétuellement scannés, attaqués par force brute et par credential stuffing, faisant d'un RDP exposé un vecteur d'accès initial de premier plan pour les rançongiciels. L'analyse de l'exposition comporte deux volets : trouver les points d'accès exposés sur internet (directement ou via des moteurs de recherche comme Shodan), et évaluer la sécurité du transport, car un hôte utilisant l'ancienne sécurité RDP native ou un TLS faible est en outre ouvert à l'attaque de la machine du milieu et au déchiffrement de session, ce qui aggrave l'exposition déjà élevée aux identifiants et à la pré-authentification.

```bash
# find exposed RDP
nmap -p3389 --open <range>
# search-engine discovery: shodan "port:3389", masscan for breadth
# assess transport
nmap -p3389 --script rdp-enum-encryption,ssl-enum-ciphers <target>
```

## Sous-thèmes

- **[Exposition sur internet](internet-exposure.md)** : trouver et évaluer un RDP exposé sur internet.
- **[TLS faible](weak-tls.md)** : sécurité RDP native et faiblesses TLS permettant le MITM.

## Références

- [CISA: RDP exposure and ransomware](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [HackTricks: RDP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
