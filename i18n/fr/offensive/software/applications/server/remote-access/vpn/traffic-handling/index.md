---
title: "Gestion du trafic : fuites VPN et failles de politique"
order: 5
description: "Même un VPN cryptographiquement solide fuit lorsque sa gestion du trafic est mal configurée : les requêtes DNS qui s'échappent du tunnel révèlent l'activité et permettent une redirection, les routes qui ne couvrent pas les destinations prévues envoient le trafic en clair, et le split tunneling relie le client entre le réseau protégé et l'internet ouvert, un chemin qu'exploite un attaquant présent sur le réseau local du client."
keywords:
  - vpn leak
  - dns leak
  - route leak
  - split tunneling
  - policy
---

# Gestion du trafic

Un VPN peut être parfaitement chiffré et échouer malgré tout par la manière dont il route et résout le trafic. Les requêtes DNS qui s'échappent du tunnel révèlent l'activité de l'utilisateur au réseau local et au résolveur configuré, et permettent à un attaquant en position d'interception de rediriger des noms. Les routes qui ne couvrent pas toutes les destinations prévues (ou qu'un attaquant local peut outrepasser) envoient en clair un trafic censé être protégé. Et le split tunneling, où seule une partie du trafic passe par le VPN, relie délibérément le client entre le réseau interne et l'internet ouvert, ce qu'un attaquant présent sur le segment local du client exploite pour atteindre le réseau interne à travers le client. Ce sont des faiblesses de configuration et de politique, pas des ruptures cryptographiques.

## Sous-sujets

- **[Fuite DNS](dns-leak.md)** : requêtes qui s'échappent du tunnel.
- **[Fuite de routage](route-leak.md)** : trafic qui contourne le tunnel par des failles de routage.
- **[Split tunneling](split-tunneling.md)** : pont entre les réseaux côté client.

## Références

- [NIST SP 800-77: VPN policy](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
- [HackTricks: VPN](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
