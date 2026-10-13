---
title: "Split tunneling : faire le pont entre les réseaux pour le client"
description: "Le split tunneling n'envoie qu'une partie du trafic par le VPN et le reste directement vers Internet, de sorte que le client est simultanément sur le réseau interne protégé et sur l'Internet ouvert. Un attaquant qui compromet le client ou se trouve sur son réseau local utilise ce pont pour atteindre le réseau interne à travers le client, déjouant le périmètre que représente le VPN."
keywords:
  - split tunneling
  - bridge
  - client compromise
  - pivot
  - perimeter bypass
---

# Split tunneling

Le split tunneling est une politique où seul le trafic destiné au réseau d'entreprise passe par le VPN tandis que tout le reste va directement vers Internet. C'est pratique et cela économise de la bande passante, mais cela fait du client un pont : en même temps, le client détient une route vers le réseau interne protégé (via le tunnel) et est exposé à son réseau local et à l'Internet ouvert (directement). Un attaquant qui compromet le client, ou se trouve sur son réseau local non fiable (domicile, café, hôtel), utilise cette double présence pour atteindre le réseau interne à travers le client, transformant un unique endpoint en pivot franchissant le périmètre du VPN que celui-ci était censé faire respecter.

```bash
# confirm split tunneling on the client: only some routes go via the tunnel
ip route                                        # internal subnets via tun, default via local gw
# an attacker on the client's local network (or who compromised the client) then:
#  - pivots through the client into the internal subnets it routes to over the tunnel
#  - reaches the client's directly-exposed services from the untrusted local side
```

## Notes d'exploitation

- Le risque est la double présence simultanée : le client route vers les ressources internes via le tunnel tout en restant joignable depuis son réseau local non fiable, de sorte que compromettre le client (depuis ce côté local ou via l'Internet qu'il touche directement) donne un pivot vers le réseau interne.
- Le mode full-tunnel (tout le trafic par le VPN) supprime ce pont ; le split tunnel est la politique exploitable, donc confirmez que le routage du client montre une route par défaut locale aux côtés de routes de tunnel internes.
- L'attaque est un pivot côté client plutôt qu'une attaque de gateway : compromettez ou faites du man-in-the-middle sur le client sur son réseau local, puis utilisez ses routes de tunnel pour atteindre l'intérieur.
- Combinez avec les [fuites de route](route-leak.md) et les [fuites DNS](dns-leak.md) ; le split tunneling est la politique qui transforme le plus directement un endpoint en point d'appui franchissant le périmètre.

## Références

- [NIST SP 800-77: tunneling policy](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
- [HackTricks: VPN](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
