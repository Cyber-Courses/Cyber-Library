---
order: 2
title: "Proxy SOCKS : accès ultérieur arbitraire via la redirection dynamique"
description: "La redirection dynamique de SSH (-D) lance un proxy SOCKS côté attaquant qui route toute connexion TCP à travers l'hôte SSH vers les réseaux qu'il peut atteindre. Combinée à proxychains, elle permet à des outils, scanners et clients arbitraires d'opérer contre un réseau interne depuis un unique point d'appui SSH, sans rediriger chaque port individuellement."
keywords:
  - proxy socks
  - ssh -D
  - redirection dynamique
  - proxychains
  - pivot
---

# Proxy SOCKS

La redirection dynamique (`-D`) transforme une connexion SSH en pivot polyvalent : au lieu de tunneliser une seule destination fixe comme la redirection locale, elle lance un proxy SOCKS sur la machine de l'attaquant qui accepte les connexions vers n'importe quelle adresse et route chacune à travers l'hôte SSH. Associée à un wrapper compatible SOCKS tel que `proxychains`, elle permet à des outils arbitraires, scanners de ports, clients web, clients de base de données, modules d'exploitation, d'opérer contre l'ensemble du réseau que l'hôte SSH peut atteindre, depuis un unique point d'appui et sans configurer une redirection par service.

```bash
# stand up a SOCKS proxy through the pivot
ssh -fN -D 1080 user@pivot
# point tools at it via proxychains (set socks5 127.0.0.1 1080 in proxychains.conf)
proxychains nmap -sT -Pn -p 22,445,3389 10.0.5.0/24
proxychains curl http://10.0.5.20/
proxychains mysql -h 10.0.5.30 -u app -p
# some tools accept a SOCKS proxy natively
curl --socks5 127.0.0.1:1080 http://10.0.5.20/
```

## Notes d'exploitation

- La redirection dynamique est le pivot efficace quand vous devez atteindre de nombreux hôtes et ports sur le réseau interne, par opposition à la redirection locale limitée à une destination par tunnel ; un seul `-D` couvre tout le réseau accessible.
- Utilisez des scans TCP connect à travers le proxy (`nmap -sT -Pn`) : SOCKS transporte du TCP, donc les scans SYN et les techniques nécessitant des sockets bruts ou de l'UDP ne fonctionnent pas à travers lui.
- `proxychains` (avec `proxy_dns` pris en compte) route la plupart des outils de façon transparente ; chaînez plusieurs proxies SOCKS pour des pivots multicouches, ou combinez avec des jump hosts.
- Le proxy hérite de l'accessibilité réseau de l'hôte SSH, il atteint donc exactement ce que cet hôte peut atteindre, ce qui explique pourquoi un pivot bien placé (un bastion, un serveur multi-homed) a tant de valeur.

## Outils

- [proxychains-ng](https://github.com/rofl0r/proxychains-ng)
- [OpenSSH -D](https://man.openbsd.org/ssh)

## Références

- [HackTricks: SOCKS and dynamic forwarding](https://book.hacktricks.xyz/generic-methodologies-and-resources/tunneling-and-port-forwarding)
