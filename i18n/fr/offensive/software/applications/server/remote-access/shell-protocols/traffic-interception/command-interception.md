---
title: "Interception de commande : lire le contenu des r-commands en transit"
description: "Les r-commands transportent la commande exécutée, ses arguments et toute sa sortie en clair. Un attaquant bien positionné les reconstruit à partir du trafic, capturant ce qui a été exécuté, les résultats, et toute donnée sensible ou identifiant secondaire manipulé pendant la session, livrant souvent davantage que la connexion elle-même."
keywords:
  - command interception
  - cleartext
  - output capture
  - rsh
  - sniffing
---

# Interception de commande

Chaque échange r-command transporte sa charge utile en clair : pour rsh/rexec la commande, ses arguments et leur sortie, et pour rlogin toute la session interactive. Un attaquant bien positionné reconstruit l'ensemble, apprenant exactement ce qui a été exécuté, voyant les résultats, et capturant toute donnée sensible ou identifiant secondaire qui transite (une commande qui affiche une configuration avec des secrets, un mot de passe `su`/`sudo` tapé dans une session rlogin, des données lues depuis des fichiers). Cela livre fréquemment davantage qu'une connexion capturée, car les commandes administratives et leur sortie exposent les systèmes et secrets avec lesquels l'opérateur travaillait.

```bash
# capture and reconstruct the command/output content
tcpdump -i eth0 -A 'port 513 or port 514' -w content.pcap
tshark -r content.pcap -q -z follow,tcp,ascii,0     # command (c->s) and output (s->c)
```

## Notes d'exploitation

- La capture de contenu bat souvent la connexion : les commandes exécutées et leur sortie révèlent la configuration, les secrets affichés sur le terminal et les identifiants secondaires saisis en milieu de session.
- Pour rsh/rexec la direction clé est du client vers le serveur (la commande) et du serveur vers le client (la sortie) ; pour rlogin reconstruisez toute la session bidirectionnelle.
- C'est passif et discret ; associez avec le [sniffing de mot de passe](password-sniffing.md) depuis la même capture pour les identifiants plus le contenu, et avec le [rejeu de session](session-replay.md) ou le détournement pour un usage actif.
- Nécessite une position on-path/tap ; les segments r-command hérités sont généralement plats et facilement interceptés.

## Références

- [HackTricks: r-commands](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rsh)
- [RFC 1282 (rlogin)](https://datatracker.ietf.org/doc/html/rfc1282)
