---
title: "Énumération : lire un réseau IRC depuis une session enregistrée"
description: "Une fois le handshake NICK/USER terminé, une session IRC normale peut lire l'intégralité du réseau : l'ircd et sa version via VERSION, le nombre d'utilisateurs et de salons via LUSERS, la liste des salons via LIST, les détails des utilisateurs via WHO et WHOIS, les membres d'un salon via NAMES, les internes du serveur via STATS, et la topologie du réseau via MAP, ainsi que l'état des services via les requêtes NickServ et ChanServ."
keywords:
  - IRC enumeration
  - LIST
  - WHOIS
  - VERSION
  - ircd version
---

# Énumération

Une session IRC enregistrée est une interface de lecture complète du réseau. Il n'existe pas d'authentification séparée pour la plupart des requêtes : après le handshake `NICK`/`USER`, le serveur répond par des réponses numériques à une poignée de commandes qui divulguent la version de l'ircd, chaque salon et son sujet, chaque utilisateur connecté et son hôte, les liens entre serveurs, et les services utilisés. La version de l'ircd obtenue via `VERSION` est la donnée la plus importante, car elle détermine si une faille serveur connue s'applique, et la liste des salons et des utilisateurs cartographie les cibles humaines et les éventuels bots.

## Prérequis

Un handshake terminé (numérique `001`). Si le serveur a envoyé `464`, fournissez d'abord le mot de passe de connexion avec `PASS <pass>` avant `NICK`/`USER`. Certains réseaux masquent la sortie de `LIST` ou les hôtes des utilisateurs tant que vous n'avez pas rejoint un salon ou défini un mode, et les services ne répondent que si NickServ/ChanServ sont réellement actifs.

## Session détaillée

```text
$ nc irc.target.lan 6667
NICK recon
USER recon 0 * :recon
:irc.target.lan 001 recon :Welcome to the TargetNet IRC Network recon
:irc.target.lan 004 irc.target.lan UnrealIRCd-6.1.0 iowrsxzdHtIRcaAqOWS ...
VERSION
:irc.target.lan 351 recon UnrealIRCd-6.1.0. irc.target.lan :build details
LUSERS
:irc.target.lan 251 recon :There are 42 users and 3 invisible on 2 servers
:irc.target.lan 252 recon 1 :operator(s) online
LIST
:irc.target.lan 322 recon #ops 4 :deployment keys here, do not paste outside
:irc.target.lan 322 recon #general 31 :TargetNet general chat
:irc.target.lan 323 recon :End of /LIST
WHO *
:irc.target.lan 352 recon #ops ~jdoe 10.0.0.5 irc.target.lan jdoe H :0 John Doe
NAMES #ops
:irc.target.lan 353 recon = #ops :@admin jdoe backupbot
WHOIS admin
:irc.target.lan 311 recon admin ~admin 10.0.0.2 * :Site Admin
:irc.target.lan 313 recon admin :is an IRC Operator
```

Lecture des numériques : `351` donne la version exacte de l'ircd (à transmettre directement à l'[exploitation serveur](server-exploitation.md)) ; `251`/`252` comptent les utilisateurs et les opérateurs ; chaque `322` correspond à un salon avec son nombre de membres et son sujet (les sujets fuitent régulièrement des secrets, comme le fait `#ops` ici) ; `352` de `WHO` donne l'ident et l'IP réelle de chaque utilisateur (`10.0.0.5`), ce qui désanonymise les opérateurs et les bots ; `353` de `NAMES` liste les membres du salon, où `@` marque un opérateur de salon ; `311` et `313` de `WHOIS` confirment qu'`admin` est un opérateur IRC et exposent son hôte.

Internes et topologie du serveur :

```text
MAP
:irc.target.lan 015 recon :irc.target.lan (42 clients)
:irc.target.lan 015 recon :  `- services.target.lan (0 clients)
STATS o
:irc.target.lan 243 recon O *@10.0.0.2 * admin 0 :operator block
```

`MAP` révèle les serveurs liés (y compris le pseudo-serveur `services.`, confirmant des services Anope/Atheme), et `STATS o` là où c'est permis vide les O:lines, qui montrent exactement quels masques d'hôte et noms d'opérateur peuvent exécuter `OPER`, désignant votre cible pour l'escalade vers le statut d'opérateur.

Interroger les services directement :

```text
PRIVMSG NickServ :INFO admin
PRIVMSG ChanServ :INFO #ops
PRIVMSG ChanServ :ACCESS #ops LIST
```

`NickServ INFO` indique si un nick est enregistré, quand il a été vu pour la dernière fois, et parfois l'adresse e-mail enregistrée ; `ChanServ INFO`/`ACCESS LIST` montre les fondateurs du salon et la liste d'accès, désignant qui usurper ou prendre le contrôle.

## Notes d'exploitation

- Enregistrez la version `004`/`351` mot pour mot avant toute autre chose ; c'est le pivot vers l'[exploitation serveur](server-exploitation.md).
- `WHO *` et `WHOIS` fuitent les IP réelles des clients à moins que le réseau n'impose un masquage d'hôte, vous donnant des adresses internes vers lesquelles pivoter sans même toucher au chat.
- Les sujets de salon et les salons de type `#ops` sont les endroits où les opérateurs collent des clés, des jetons et des commandes de bot ; rejoignez en lecture seule et faites défiler l'historique.
- Un nœud `services.` dans `MAP` plus des réponses de NickServ/ChanServ vous indique qu'une prise de contrôle de compte est possible ; passez à [abus des services et des opérateurs](services-and-operator-abuse.md).

## Références

- [RFC 1459 (IRC, numeric replies)](https://datatracker.ietf.org/doc/html/rfc1459#section-6)
- [RFC 2812 (IRC Client Protocol)](https://datatracker.ietf.org/doc/html/rfc2812)
- [Nmap irc-info script](https://nmap.org/nsedoc/scripts/irc-info.html)
