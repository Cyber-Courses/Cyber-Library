---
title: "Fuite de route : trafic contournant le VPN via des lacunes de routage"
description: "Un VPN ne protège que le trafic que ses routes envoient dans le tunnel. Quand les routes poussées ne couvrent pas toutes les destinations prévues, quand l'IPv6 n'est pas routé sur un tunnel IPv4, ou quand un attaquant local injecte une route plus spécifique, le trafic destiné au tunnel sort en clair, l'exposant au réseau local alors que le VPN est actif."
keywords:
  - route leak
  - routing table
  - ipv6 leak
  - more specific route
  - tunnel bypass
---

# Fuite de route

Un VPN ne protège que le trafic que son routage envoie dans le tunnel ; tout ce que la table de routage dirige ailleurs part en clair. Les fuites de route surviennent de plusieurs façons : le serveur pousse des routes qui ne couvrent pas réellement toutes les destinations que l'utilisateur croit protégées ; le trafic IPv6 est laissé sur l'interface native quand le tunnel ne capte que l'IPv4 (de sorte que les destinations dual-stack fuient sur IPv6) ; ou un attaquant local injecte une route plus spécifique (via de faux router advertisements ou DHCP) qui l'emporte sur la route du VPN, tirant le trafic ciblé hors du tunnel. Dans chaque cas, du trafic censé être protégé est exposé au réseau local alors que le VPN semble actif.

```bash
# inspect the client routing table to see what is (not) tunnelled
ip route; ip -6 route                           # does the default/intended route go via the tun?
#   a native-interface route for a destination means it bypasses the tunnel
# local attacker: inject a more-specific route to pull traffic out of the tunnel
#   rogue RA (IPv6) or DHCP option to add a competing, more-specific route on the client
```

## Notes d'exploitation

- L'IPv6 est la fuite classique : un tunnel IPv4 uniquement sur un client dual-stack envoie le trafic IPv6 nativement, de sorte qu'un attaquant offrant une connectivité IPv6 (de faux router advertisements) capte ce trafic en dehors du VPN.
- Le routage par longest-prefix-match signifie qu'une route plus spécifique injectée par un attaquant local (faux RA/DHCP) bat la route plus large du VPN pour la destination ciblée, la tirant en clair, un contournement sélectif.
- Des routes poussées incomplètes exposent tout ce qu'elles omettent ; comparez la table de routage du client à ce que l'utilisateur suppose protégé.
- Les fuites de route exposent le contenu du trafic (pas seulement le DNS) ; combinez avec une position de capture et avec la [fuite DNS](dns-leak.md) et le [split tunneling](split-tunneling.md) pour le tableau complet de la gestion du trafic.

## Références

- [IPv6 VPN leakage](https://www.rfc-editor.org/rfc/rfc7359)
- [HackTricks: VPN leaks](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
