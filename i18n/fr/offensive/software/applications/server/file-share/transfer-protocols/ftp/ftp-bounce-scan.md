---
title: "FTP bounce scan : relayer des scans de ports à travers le serveur"
description: "La commande FTP PORT permet à un client de spécifier l'adresse de la connexion de données, et un serveur qui ne la restreint pas à l'adresse du client peut recevoir l'ordre d'ouvrir des connexions de données vers des hôtes et des ports arbitraires. Ce rebond transforme le serveur FTP en proxy de scan, atteignant des hôtes et des ports internes que l'attaquant ne peut pas joindre directement."
keywords:
  - ftp bounce
  - port command
  - scan proxy
  - réseau interne
  - nmap -b
---

# Bounce scan

La connexion de données FTP en mode actif est établie avec la commande `PORT`, par laquelle le client indique au serveur l'IP et le port auxquels se connecter pour le transfert. Le protocole n'exige pas que cette adresse soit celle du client, de sorte qu'un serveur qui ne parvient pas à restreindre `PORT` au pair de la connexion de contrôle peut recevoir l'instruction d'ouvrir des connexions de données vers n'importe quel hôte et port. En observant si la connexion de données réussit, un attaquant utilise le serveur FTP comme proxy pour scanner des hôtes et des ports qu'il peut joindre mais que l'attaquant ne peut pas, typiquement des systèmes internes situés derrière le serveur FTP.

```bash
# nmap's built-in FTP bounce scan: scan <target> via the FTP relay
nmap -b anonymous:a@<ftp-relay> -p 1-1024 <internal-target>
# the relay (ftp-relay) issues PORT to <internal-target>; open vs refused is inferred
# manual: PORT a,b,c,d,p1,p2 then LIST; the server's response reveals reachability
```

Le scan déduit l'état d'un port à partir de la réponse du serveur FTP à une commande de transfert de données après un `PORT` forgé : une connexion de données réussie implique que le port cible est ouvert, un refus implique qu'il est fermé ou filtré. Cela atteint le réseau interne depuis le point de vue du serveur FTP.

## Notes d'exploitation

- L'intérêt est le pivotement : le serveur FTP scanne des hôtes et des ports internes injoignables depuis l'attaquant, cartographiant un segment de réseau situé derrière lui sans aucun point d'appui sur le serveur lui-même.
- Il faut un serveur qui ne lie pas les cibles `PORT` à l'adresse du client ; les serveurs modernes restreignent généralement cela, on le trouve donc surtout sur de vieux démons FTP ou des démons mal configurés.
- Au-delà du scan, le rebond peut dans certains cas livrer des octets à un service interne (un proxy d'écriture de données limité), bien que la cartographie des ports en soit l'usage courant.
- `nmap -b` l'automatise ; l'accès `anonymous` au relais suffit lorsque l'accès anonyme est autorisé.

## Références

- [nmap FTP bounce (-b)](https://nmap.org/book/scan-methods-ftp-bounce-scan.html)
- [RFC 959 : commande PORT](https://datatracker.ietf.org/doc/html/rfc959)
