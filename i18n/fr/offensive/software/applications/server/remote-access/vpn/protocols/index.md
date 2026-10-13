---
order: 4
title: "Protocoles : attaquer des protocoles VPN spécifiques"
description: "Les protocoles VPN diffèrent nettement par leurs faiblesses : IPsec/IKE expose la divulgation de PSK en mode agressif, la sécurité d'OpenVPN repose sur la configuration et la gestion des clés, le MS-CHAPv2 de PPTP est cryptographiquement cassé, WireGuard est solide mais fuit via la gestion des clés statiques, et L2TP et SSTP héritent respectivement des faiblesses de la PSK IPsec et de TLS. Le protocole en usage dicte l'attaque."
keywords:
  - ipsec
  - ike
  - openvpn
  - pptp
  - wireguard
---

# Protocoles

Les VPN fonctionnent sur différents protocoles, et chacun porte ses propres faiblesses caractéristiques, de sorte qu'identifier le protocole sélectionne l'attaque. IPsec avec IKE expose la divulgation du hash de clé pré-partagée en mode agressif et des faiblesses de transformation. OpenVPN est cryptographiquement solide mais sa sécurité dépend entièrement de la configuration et de la gestion des clés/certificats. PPTP est obsolète et son authentification MS-CHAPv2 est de fait cassée. Le protocole de WireGuard est solide, de sorte que l'attaque se déplace vers sa gestion des clés statiques et l'exposition d'identité. Et L2TP et SSTP héritent des faiblesses de ce qui les encapsule, une clé pré-partagée IPsec et TLS respectivement.

## Sous-thèmes

- **[IPsec IKE](ipsec-ike.md)** : divulgation de PSK en mode agressif et faiblesses IKE.
- **[OpenVPN](openvpn.md)** : attaques de configuration, d'informations d'identification et de gestion des clés.
- **[PPTP](pptp.md)** : l'authentification MS-CHAPv2 cassée.
- **[WireGuard](wireguard.md)** : gestion des clés statiques et exposition d'identité.
- **[L2TP et SSTP](l2tp-and-sstp.md)** : faiblesses héritées de la PSK IPsec et de TLS.

## Références

- [NIST SP 800-77 (protocoles VPN)](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
- [HackTricks: IPsec/IKE](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
