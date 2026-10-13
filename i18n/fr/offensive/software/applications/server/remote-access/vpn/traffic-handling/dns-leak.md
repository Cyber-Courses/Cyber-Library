---
title: "Fuite DNS : requêtes échappant au tunnel VPN"
description: "Lorsqu'un client VPN envoie des requêtes DNS en dehors du tunnel, vers le résolveur local ou celui du FAI plutôt que celui du VPN, il divulgue les noms que l'utilisateur résout et laisse un attaquant local ou sur le chemin voir et rediriger cette activité. Les fuites DNS révèlent des noms d'hôte internes et le comportement de l'utilisateur et permettent une redirection fondée sur les noms malgré le tunnel."
keywords:
  - dns leak
  - resolver
  - tunnel
  - redirection
  - leak test
---

# Fuite DNS

Une fuite DNS, c'est lorsque le client VPN résout des noms via un résolveur en dehors du tunnel, celui du réseau local ou du FAI, au lieu du résolveur fourni par le VPN, généralement parce que le client ne parvient pas à remplacer le résolveur système ou que l'OS bascule en secours. La conséquence est une divulgation : quiconque voit ces requêtes (le réseau local, le résolveur fuité) apprend les noms que l'utilisateur consulte, ce qui révèle les noms d'hôte internes qu'il atteint et son activité générale, déjouant une partie de la confidentialité du VPN. Cela permet aussi une redirection : un attaquant sur le chemin qui répond aux requêtes fuitées oriente l'utilisateur vers des adresses contrôlées par l'attaquant malgré le VPN.

```bash
# from the client's perspective, check which resolver is actually used on the tunnel
resolvectl status                               # per-link DNS; is the VPN resolver authoritative?
# observe leaked queries from the local network (on-path)
tcpdump -i eth0 -n udp port 53                  # queries going out the local interface, not the tunnel
# public leak-test services confirm which resolver the client appears to use
```

## Notes d'exploitation

- Pour un attaquant sur le réseau local du client, les requêtes fuitées sont à la fois du renseignement (les noms et services internes que l'utilisateur atteint via le VPN) et une opportunité : répondre au DNS fuité pour rediriger l'utilisateur vers l'infrastructure de l'attaquant.
- Les noms d'hôte internes dans les requêtes fuitées cartographient le nommage et les services du réseau protégé, une reconnaissance utile même sans redirection.
- Les fuites surviennent couramment parce que le client ne se définit pas comme résolveur faisant autorité, que le DNS IPv6 échappe à un tunnel IPv4 uniquement, ou que l'OS bascule en secours sur un autre résolveur ; vérifiez le DNS par lien sur le client.
- Cela se combine avec la [fuite de route](route-leak.md) et le [split tunneling](split-tunneling.md) comme lacunes de gestion de trafic qu'un attaquant local exploite autour d'un tunnel par ailleurs sain.

## Références

- [DNS leak background](https://www.dnsleaktest.com/what-is-a-dns-leak.html)
- [HackTricks: VPN leaks](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
