---
order: 4
title: "WireGuard : gestion des clés statiques et exposition d'identité"
description: "La cryptographie de WireGuard est moderne et solide, de sorte que les attaques visent sa gestion des clés : récupérer les clés privées statiques et les clés pré-partagées dans les fichiers de configuration client et serveur, où elles se trouvent en clair, et l'exposition d'identité issue de son modèle de clé fixe par pair. Une clé récupérée authentifie l'attaquant en tant que ce pair."
keywords:
  - wireguard
  - clé statique
  - clé pré-partagée
  - wg0.conf
  - gestion des clés
---

# WireGuard

WireGuard est délibérément minimal et cryptographiquement solide, de sorte qu'il n'existe pas d'attaque cryptographique au niveau du protocole du type de celles qui affligent PPTP ou l'IKE en mode agressif ; il reste même silencieux face aux sondes non authentifiées. La surface d'attaque est donc la gestion des clés. WireGuard authentifie les pairs avec des paires de clés statiques Curve25519 et une clé pré-partagée optionnelle, et celles-ci résident en clair dans les fichiers de configuration (`wg0.conf`, les fichiers `.conf` clients) et dans la configuration en cours d'exécution. Récupérer la clé privée d'un pair (et la PSK, si utilisée) depuis une configuration, une sauvegarde ou un appareil permet à l'attaquant de monter un pair avec cette identité et de se connecter. Le modèle à clé fixe fait aussi que la clé publique d'un pair est une identité persistante qui révèle qui se connecte.

```bash
# recover keys from configuration (cleartext)
cat /etc/wireguard/wg0.conf                      # PrivateKey, PresharedKey, peer PublicKeys
grep -rE 'PrivateKey|PresharedKey' /etc/wireguard /mnt/loot ~/ 2>/dev/null
# the running config also exposes them
wg showconf wg0
# with a peer's PrivateKey (+ PSK if set), configure a client to connect as that peer
```

## Notes d'exploitation

- Toute l'attaque consiste à trouver la clé privée statique (et la PSK) : elles sont stockées en clair dans `wg0.conf`/les configurations clientes et visibles via `wg showconf`, de sorte qu'une configuration lisible, une sauvegarde ou un terminal compromis donne une identité utilisable.
- Avec la clé privée d'un pair et l'endpoint/allowed-IPs correspondants (également dans la configuration), l'attaquant monte un pair et se connecte en tant que cette identité ; aucun cassage n'est impliqué car le protocole est solide.
- La clé pré-partagée optionnelle ajoute un facteur symétrique ; elle aussi réside dans la configuration, de sorte que récupérer la configuration récupère généralement les deux.
- Protéger la configuration est le véritable contrôle ; traitez le matériel de clés WireGuard comme n'importe quelle clé privée de grande valeur et cherchez-le en conséquence, à l'image de l'[abus de certificat](../authentication/certificate-abuse.md).

## Références

- [Protocole et configuration WireGuard](https://www.wireguard.com/)
- [wg(8) et wg-quick](https://man7.org/linux/man-pages/man8/wg.8.html)
