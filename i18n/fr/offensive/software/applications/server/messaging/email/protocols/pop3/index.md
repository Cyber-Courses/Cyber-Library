---
title: "POP3 : attaquer la récupération de boîte aux lettres en téléchargement seul"
order: 3
description: "Attaquer POP3, le protocole de récupération de courrier en téléchargement puis suppression sur le port 110 (clair/STARTTLS) et 995 (TLS implicite). Couvre l'énumération par bannière et CAPA, la validation des noms d'utilisateur via USER, l'authentification en clair USER/PASS et APOP, le téléchargement de la boîte aux lettres avec RETR et TOP, et les failles serveur dans qpopper, Courier et Dovecot."
keywords:
  - pop3
  - 995
  - apop
  - retr
  - qpopper
---

# POP3

POP3 est le cousin simple d'IMAP : il existe pour télécharger les messages d'une boîte aux lettres unique et, classiquement, les supprimer du serveur au fur et à mesure. Il écoute sur le port 110 (en clair, parfois avec STARTTLS) et 995 (TLS implicite). Le protocole est un ensemble plat de commandes courtes avec des réponses `+OK`/`-ERR`, sans modèle de dossiers, sans recherche côté serveur et avec un état minimal. Cette simplicité est offensivement commode : un identifiant valide et une poignée de commandes vident toute la boîte aux lettres. Les deux mêmes directions s'appliquent que pour tout service de messagerie : lire le courrier pour le butin et réutiliser l'identifiant, plus les failles propres à l'implémentation du serveur (qpopper sur les hôtes hérités, Courier, le service POP3 de Dovecot).

## Trouver et router le service

```bash
nmap -p110,995 -sV --script pop3-capabilities,pop3-ntlm-info <target>
# pop3-capabilities lists STLS (STARTTLS), SASL, USER, UIDL, APOP
# pop3-ntlm-info leaks domain/host from an NTLM challenge
openssl s_client -connect <target>:995 -quiet     # implicit-TLS session
USER alice
PASS Sprint2025!
```

Chaque réponse est soit `+OK` (succès, souvent avec des données à suivre) soit `-ERR` (échec avec une chaîne de raison). Contrairement à IMAP, il n'y a pas d'étiquettes de commande ; les réponses sont positionnelles, vous les lisez donc dans l'ordre. Comme POP3 supprime fréquemment le courrier lors de la récupération, préférez `TOP` pour jeter un œil avant de vous engager dans un `RETR` sur une boîte aux lettres que vous devez laisser intacte.

## Pages

- **[Énumération](enumeration.md)** : bannière du message d'accueil, `CAPA`, présence d'APOP et validation des noms d'utilisateur via `USER`.
- **[Authentification](authentication.md)** : `USER`/`PASS` en clair, le défi-réponse APOP et la pulvérisation.
- **[Accès à la boîte aux lettres](mailbox-access.md)** : `STAT`/`LIST`/`RETR`/`TOP` pour télécharger et piller sans détruire la boîte aux lettres.
- **[Exploitation du serveur](server-exploitation.md)** : failles d'implémentation POP3 de qpopper, Courier et Dovecot et leur lignée.

## Références

- [RFC 1939 : Post Office Protocol v3](https://datatracker.ietf.org/doc/html/rfc1939)
- [HackTricks : pentesting POP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-pop.html)
- [nmap pop3-capabilities](https://nmap.org/nsedoc/scripts/pop3-capabilities.html)
