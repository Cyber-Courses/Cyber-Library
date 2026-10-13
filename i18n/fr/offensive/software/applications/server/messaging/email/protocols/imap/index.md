---
title: "IMAP : attaquer l'accès à boîte aux lettres avec état"
order: 2
description: "Attaquer IMAP, le protocole d'accès à boîte aux lettres avec état sur 143 (clair/STARTTLS) et 993 (TLS implicite). Couvre l'énumération des capacités et des utilisateurs, l'authentification en clair et SASL, la lecture et la recherche de butin dans la boîte aux lettres après authentification, et les failles d'implémentation serveur dans Dovecot, Cyrus et Courier."
keywords:
  - imap
  - 993
  - dovecot
  - mailbox access
  - mail harvesting
---

# IMAP

IMAP conserve le courrier sur le serveur et permet à un client de travailler dessus sur place : lister les dossiers, sélectionner une boîte aux lettres, chercher et récupérer des messages individuels ou des en-têtes. Il écoute sur 143 (en clair, souvent avec STARTTLS) et 993 (TLS implicite), et contrairement à POP3 il est avec état et étiqueté, de sorte que chaque commande client porte une étiquette que le serveur reprend dans sa réponse finale. La valeur offensive va dans deux directions : des identifiants valides se transforment en lecture complète de la boîte aux lettres (identifiants, liens de réinitialisation de mot de passe, documents internes, chaînes de réponses pour le hameçonnage), et l'implémentation serveur elle-même (couramment Dovecot, parfois Cyrus ou Courier) véhicule ses propres failles.

## Trouver et orienter le service

```bash
nmap -p143,993 -sV --script imap-capabilities,imap-ntlm-info <target>
# capabilities reveal STARTTLS, LOGINDISABLED, and AUTH= mechanisms
# imap-ntlm-info leaks the AD/NetBIOS/domain/FQDN from an NTLM challenge
openssl s_client -connect <target>:993 -quiet      # implicit-TLS session
a LOGIN user pass                                  # tagged login once connected
```

Une réponse étiquetée `a OK` signifie que la commande a réussi, `a NO` un échec propre (mauvais identifiants, permission refusée), et `a BAD` une erreur de protocole/syntaxe. Sur 143, exécutez d'abord `A1 CAPABILITY` : `LOGINDISABLED` signifie que le `LOGIN` en clair est refusé jusqu'à ce que STARTTLS soit négocié, et les jetons `AUTH=` listent les mécanismes SASL disponibles pour le spraying.

## Pages

- **[Énumération](enumeration.md)** : listage des capacités, empreinte d'implémentation basée sur la bannière, et validation des noms d'utilisateur par différenciation des réponses.
- **[Authentification](authentication.md)** : `LOGIN` en clair, mécanismes SASL `AUTHENTICATE`, spraying, et ce que change `LOGINDISABLED`.
- **[Accès à la boîte aux lettres](mailbox-access.md)** : `LIST`/`SELECT`/`SEARCH`/`FETCH` après authentification pour lire et piller le courrier, et export en masse.
- **[Exploitation serveur](server-exploitation.md)** : failles d'implémentation de Dovecot, Cyrus et Courier et leur filiation.

## Références

- [RFC 3501 : IMAP4rev1](https://datatracker.ietf.org/doc/html/rfc3501)
- [HackTricks : pentesting IMAP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-imap.html)
- [Documentation Dovecot](https://doc.dovecot.org/)
