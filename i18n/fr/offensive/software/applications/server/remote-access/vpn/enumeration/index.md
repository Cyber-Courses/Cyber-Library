---
order: 3
title: "Énumération : fingerprinting d'un point de terminaison VPN"
description: "L'énumération VPN identifie quel protocole et quel produit VPN une passerelle exécute, à partir de la négociation IKE UDP (IPsec), du port de contrôle PPTP, ou du portail web SSL-VPN, et en extrait des détails de version et de configuration. Cette identification sélectionne l'attaque de protocole et, pour les appliances, relie le build exact à ses exploits pré-authentification."
keywords:
  - énumération vpn
  - ike-scan
  - détection de protocole
  - portail
  - version
---

# Énumération

L'énumération VPN détermine d'abord ce qu'est la passerelle : la négociation IKE UDP sur 500/4500 indique IPsec, TCP 1723 indique PPTP, un portail web TLS sur 443 indique une appliance SSL-VPN (et quel éditeur), et 1194 suggère OpenVPN. Ensuite, elle extrait la version et la configuration : les ensembles de transformations IKE et le fait que le mode agressif soit proposé, le produit et le build du portail SSL-VPN (depuis la page de connexion, les en-têtes et les ressources), et les détails TLS. Cela pilote tout, le protocole détermine l'attaque, et pour les appliances le build exact se relie à son exploit pré-authentification.

```bash
nmap -sU -p500,4500 --script ike-version <target>    # IPsec/IKE
ike-scan -M <target>; ike-scan -A -M <target>        # main vs aggressive mode transforms
nmap -p1723 --script pptp-version <target>           # PPTP
curl -skI https://<target>/                          # SSL-VPN portal product/headers
```

## Sous-thèmes

- **[Détection de protocole](protocol-detection.md)** : identifier le protocole VPN et les ports.
- **[Récupération de bannière](banner-grabbing.md)** : divulgation de version et de produit.

## Références

- [ike-scan](https://github.com/royhills/ike-scan)
- [HackTricks: IPsec/IKE enumeration](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
