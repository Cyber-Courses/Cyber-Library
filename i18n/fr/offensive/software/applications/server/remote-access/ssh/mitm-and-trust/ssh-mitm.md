---
title: "SSH MITM : s'interposer pour capturer les identifiants et détourner la session"
description: "Avec une position réseau et une vérification de clé d'hôte vaincue, un attaquant exécute un proxy SSH machine-in-the-middle qui termine la connexion du client avec une clé d'hôte substituée et relaie vers le vrai serveur. Cela capture le mot de passe ou le matériel de clé que le client soumet, enregistre la session en clair et permet d'y injecter des commandes."
keywords:
  - mitm ssh
  - substitution de clé d'hôte
  - capture d'identifiants
  - détournement de session
  - ssh-mitm
---

# SSH MITM

Un machine-in-the-middle SSH combine une position réseau avec une vérification de clé d'hôte vaincue pour se placer entre le client et le serveur. Le proxy de l'attaquant accepte la connexion du client en présentant sa propre clé d'hôte (approuvée grâce à un [contournement de known_hosts](known-hosts-bypass.md)), authentifie le client contre lui-même, et relaie vers le vrai serveur. Parce que le proxy déchiffre et re-chiffre, il voit tout ce que le client envoie : le mot de passe dans une connexion par mot de passe, ou, pour l'auth par clé, la possibilité d'utiliser la clé transmise par l'agent du client ou de capturer la session. Il enregistre toute la session en clair et peut injecter des commandes que l'utilisateur n'a jamais tapées.

```bash
# 1. gain an on-path position to the target SSH (ARP spoof, rogue DNS, route)
arpspoof -i eth0 -t <client> <gateway>; sysctl -w net.ipv4.ip_forward=1
# 2. run an SSH MITM proxy that presents a substitute host key and relays
ssh-mitm server --remote-host <real-server>    # captures creds + logs session
# 3. the client connects (to the attacker), is prompted/accepts the host key,
#    submits its password (captured) or forwards its agent (abusable)
```

## Notes d'exploitation

- Les connexions par mot de passe sont entièrement capturées : le proxy voit le mot de passe en clair que le client envoie, livrant directement l'identifiant.
- Pour l'auth à clé publique, la clé privée n'est pas transmise, mais si le client active la transmission d'agent, le MITM peut utiliser l'agent transmis pour s'authentifier plus loin en tant que l'utilisateur tant que la session est active ; sans transmission, le proxy enregistre quand même et peut injecter dans la session.
- Toute l'attaque dépend de l'échec de la vérification de clé d'hôte (première utilisation, vérification stricte désactivée, ou un `known_hosts` pré-empoisonné) ; contre un client qui épingle et applique la clé, la clé substituée déclenche un échec franc.
- La crypto négociée faible ([chiffrements faibles](../weak-cryptography/weak-ciphers.md)) est une voie distincte pour attaquer une session capturée, mais le MITM de clé d'hôte est la voie directe de capture d'identifiants.

## Outils

- [ssh-mitm](https://github.com/ssh-mitm/ssh-mitm)
- [dsniff/arpspoof (positioning)](https://www.monkey.org/~dugsong/dsniff/)

## Références

- [ssh-mitm documentation](https://docs.ssh-mitm.at/)
- [HackTricks: SSH MITM](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
