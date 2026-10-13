---
title: "Force brute de mot de passe : attaquer le schéma de mot de passe VNC"
description: "L'authentification VNC classique est un challenge-response DES sur un mot de passe tronqué à huit caractères. Elle est forcée par force brute en ligne contre le serveur, et, plus puissamment, cassée hors ligne à partir d'une seule paire challenge-response capturée, car l'espace de mots de passe court et l'algorithme connu rendent la récupération hors ligne rapide."
keywords:
  - force brute vnc
  - challenge des
  - cassage hors ligne
  - huit caractères
  - auth rfb
---

# Force brute de mot de passe

L'authentification standard de VNC (type de sécurité 2) est un challenge-response : le serveur envoie un challenge de 16 octets, le client le chiffre avec DES clé par le mot de passe (tronqué à huit caractères, avec une particularité qui inverse l'ordre des bits de chaque octet de clé), et renvoie le résultat. Deux attaques en découlent. La force brute en ligne essaie des mots de passe directement contre le serveur. Plus puissamment, capturer une seule paire challenge-response (par sniffing ou MITM) permet un cassage hors ligne : l'espace de mots de passe est minuscule (huit caractères, et l'algorithme est connu), de sorte qu'une wordlist ou une force brute récupère le mot de passe rapidement sans retoucher le serveur.

```bash
# online brute force
nmap -p5900 --script vnc-brute --script-args passdb=rockyou.txt <target>
hydra -P rockyou.txt vnc://<target>
# offline: capture the challenge + response (sniff/MITM), then crack
#   VNC auth is a known DES-challenge format; feed challenge+response to a cracker
#   (the 8-char cap and reversed-bit key make the keyspace small and fast to search)
```

## Notes d'exploitation

- La troncature à huit caractères est la faiblesse décisive : seuls les huit premiers caractères sont pris dans la clé, de sorte que l'espace effectif est petit et les wordlists très efficaces ; les mots de passe plus longs n'apportent rien au-delà de huit octets.
- Le cassage hors ligne à partir d'un challenge-response capturé est discret et rapide, et évite le verrouillage/journalisation en ligne ; une seule authentification sniffée (le trafic est par ailleurs en clair) donne la paire, voir [transmission en clair](../weak-cryptography/cleartext-transmission.md).
- La force brute en ligne fonctionne là où vous ne pouvez pas capturer, mais les serveurs VNC peuvent limiter ou retarder les tentatives ; gardez les listes courtes étant donné le plafond de 8 caractères.
- Un mot de passe cassé est réutilisable entre points de terminaison VNC et parfois d'autres services ; l'accès au bureau récupéré est un contrôle interactif de la cible.

## Outils

- [nmap vnc-brute](https://nmap.org/nsedoc/scripts/vnc-brute.html)
- [hydra](https://github.com/vanhauser-thc/thc-hydra)

## Références

- [RFC 6143: VNC authentication](https://datatracker.ietf.org/doc/html/rfc6143#section-7.2.2)
- [HackTricks: VNC brute force](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
