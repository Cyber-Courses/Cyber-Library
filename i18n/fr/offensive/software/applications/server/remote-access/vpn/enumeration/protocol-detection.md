---
title: "Détection de protocole : identifier le protocole VPN et les ports"
description: "Différentes technologies VPN écoutent sur des ports caractéristiques et répondent à des sondes distinctes : IPsec/IKE sur UDP 500 et 4500, PPTP sur TCP 1723, OpenVPN généralement sur 1194, SSTP et les portails SSL-VPN sur TCP 443, et WireGuard sur un port UDP. Identifier le protocole sélectionne l'attaque et l'outillage applicables."
keywords:
  - détection de protocole
  - port 500
  - port 1723
  - openvpn
  - sslvpn
---

# Détection de protocole

Chaque technologie VPN a une empreinte reconnaissable, et l'identifier est la première étape d'énumération parce que le protocole dicte l'attaque. IPsec utilise UDP 500 pour IKE et 4500 pour le NAT-traversal ; PPTP utilise TCP 1723 (avec GRE) ; OpenVPN utilise par défaut UDP ou TCP 1194 ; SSTP et les portails SSL-VPN empruntent TCP 443 (distingués par la réponse HTTP et le produit) ; et WireGuard écoute sur un port UDP configurable, ne répondant qu'à une négociation valide (silencieux sinon). Un scan UDP plus des sondes ciblées relient la passerelle à son protocole.

```bash
# scan the characteristic ports
nmap -sU -p500,4500,1194,51820 <target>        # IKE, NAT-T, OpenVPN(UDP), WireGuard
nmap -sT -p1723,443,1194 -sV <target>          # PPTP, SSL-VPN/SSTP, OpenVPN(TCP)
ike-scan <target>                               # confirms IPsec/IKE by handshake
# OpenVPN and WireGuard are often silent to non-protocol probes; a valid handshake confirms
```

## Notes d'exploitation

- Le port 500/4500 UDP répondant à `ike-scan` confirme IPsec et ouvre les attaques [IKE](../protocols/ipsec-ike.md) (mode agressif, PSK) ; 1723 confirme [PPTP](../protocols/pptp.md) ; 443 avec un portail VPN pointe vers une [appliance SSL-VPN](../ssl-vpn-appliances/index.md).
- WireGuard et OpenVPN sont souvent silencieux face aux scans génériques (WireGuard ne répond qu'à une négociation cryptographiquement valide), de sorte que l'absence de bannière ne signifie pas l'absence du service ; déduisez-le du port et de toute configuration cliente.
- Sur 443, distinguez un simple portail SSL-VPN de SSTP et d'une application web générale par la réponse HTTP et les empreintes de produit, ce que la [récupération de bannière](banner-grabbing.md) affine.
- Le protocole identifié sélectionne à la fois l'attaque et l'outillage (`ike-scan` pour IPsec, exploits spécifiques au produit pour les appliances).

## Références

- [ike-scan](https://github.com/royhills/ike-scan)
- [HackTricks: VPN protocols](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
