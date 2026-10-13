---
title: "Abus des services et des opérateurs : s'emparer des nicks, des salons et du statut d'opérateur IRC"
description: "Les services IRC (NickServ, ChanServ) protègent la propriété des comptes et des salons par un mot de passe, donc les identifier ou les forcer par brute force permet de prendre le contrôle des nicks et des salons enregistrés. La commande OPER promeut une session au rang d'opérateur IRC, débloquant les commandes serveur (KILL, SAMODE, REHASH, DIE, RESTART) et, sur certains ircds, le chargement de modules et l'exécution de commandes sur l'hôte. Des O:lines faibles et SASL rendent les deux atteignables."
keywords:
  - NickServ
  - ChanServ
  - OPER
  - IRC operator
  - SASL
---

# Abus des services et des opérateurs

IRC divise le privilège en deux systèmes que vous pouvez attaquer séparément. Les services (NickServ et ChanServ, adossés à Anope ou Atheme) possèdent l'enregistrement des nicks et des salons et le protègent par un mot de passe choisi par l'utilisateur, donc deviner ou capturer ce mot de passe vous remet le compte ou le salon. Le statut d'opérateur est au niveau du serveur : la commande `OPER <name> <password>` vérifie votre session par rapport à une O:line (un bloc d'opérateur dans la configuration de l'ircd qui associe un nom et une empreinte de mot de passe à un masque d'hôte et à un ensemble de privilèges) et, en cas de succès, accorde des commandes de contrôle du réseau et, sur de nombreux démons, la capacité de charger des modules ou d'exécuter des commandes sur l'hôte. Les deux sont des problèmes de mot de passe, et les deux sont fréquemment faibles.

## Prérequis

Une session enregistrée (numérique `001`). Pour l'abus des services, NickServ/ChanServ doivent être actifs (confirmé dans l'[énumération](enumeration.md) via un nœud `services.` dans `MAP`). Pour l'escalade vers opérateur, vous avez besoin d'un nom d'opérateur candidat et d'un mot de passe ; `STATS o` là où c'est lisible nomme les O:lines et leurs masques d'hôte, et les mots de passe d'opérateur sont souvent réutilisés depuis des sujets de salon, des comptes de service ou des fuites de configuration.

## Identifier et s'emparer des comptes de service

```text
PRIVMSG NickServ :IDENTIFY admin Summer2026!
-NickServ- Password incorrect.
PRIVMSG NickServ :IDENTIFY admin CorrectHorse
-NickServ- You are now identified for admin.
PRIVMSG ChanServ :SET #ops FOUNDER recon
```

Un `IDENTIFY` réussi lie la session à ce nick enregistré ; une fois identifié comme fondateur d'un salon, vous pouvez réassigner le fondateur, vous accorder le statut d'opérateur dans le salon, ou expulser d'autres utilisateurs. SASL fait la même chose au moment de la connexion et est scriptable pour le brute force contre le port en clair :

```bash
# brute-force a NickServ account over SASL PLAIN during the handshake
# authzid\0authcid\0password, base64-encoded, sent in an AUTHENTICATE line
printf '\0admin\0CorrectHorse' | base64
# => AGFkbWluAENvcnJlY3RIb3JzZQ==
# in the handshake: CAP REQ :sasl / AUTHENTICATE PLAIN / AUTHENTICATE <base64>
# numeric 903 = SASL authentication successful; 904 = failed
```

Interprétez `903` comme un identifiant valide (le nick vous appartient à chaque reconnexion) et `904` comme un mauvais mot de passe ; itérez la ligne base64 pour une liste de mots.

## Escalade vers opérateur IRC

```text
OPER netadmin Autumn2026!
:irc.target.lan 464 recon :Password incorrect
OPER netadmin Sup3rSecret
:irc.target.lan 381 recon :You are now an IRC operator
```

Le numérique `381`, c'est tout le jeu : la session est désormais opératrice. `464` est un mauvais mot de passe (ou un masque d'hôte qui ne correspond pas à la O:line, auquel cas aucun mot de passe ne fonctionnera depuis votre adresse). Une fois à `381`, l'ensemble des commandes d'opérateur est disponible :

```text
MODE recon +o
SAMODE #ops +o recon          # force channel-op in any channel (InspIRCd/UnrealIRCd)
KILL jdoe :cleanup            # forcibly disconnect a user to seize their nick
REHASH                        # reload server config from disk
```

`SAMODE` (ou `SAJOIN`/`SANICK` sur les démons qui les proposent) permet à un opérateur de s'emparer du contrôle de n'importe quel salon indépendamment de ChanServ, et `KILL` déconnecte une cible pour que vous puissiez saisir son nick avant qu'elle ne se reconnecte. Sur UnrealIRCd et InspIRCd, un opérateur disposant des bons privilèges peut charger un module serveur, et certaines versions exposent un chemin de configuration ou un bloc de commande qui s'exécute sur l'hôte, transformant le statut d'opérateur en exécution sur la machine de l'ircd. `DIE` et `RESTART` arrêtent ou redémarrent le serveur entier.

## Suite

- La prise de contrôle d'un service ou d'un fondateur donne un contrôle durable sur des salons de grande valeur (`#ops`, salons de contrôle de bots) et la capacité d'usurper des utilisateurs de confiance auprès du reste du réseau.
- Le statut d'opérateur (`381`) est un contrôle du réseau : emparez-vous de n'importe quel salon avec `SAMODE`, désanonymisez chaque utilisateur, et sur les démons capables de charger des modules, pivotez vers l'[exploitation serveur](server-exploitation.md) et un shell en tant qu'utilisateur de l'ircd.
- Un mot de passe d'opérateur ou de service capturé est fréquemment réutilisé sur le shell de l'hôte, la base de données des services ou d'autres infrastructures ; testez-le largement.

## Références

- [RFC 2812 (IRC Client Protocol, OPER)](https://datatracker.ietf.org/doc/html/rfc2812#section-3.1.4)
- [IRCv3 SASL authentication extension](https://ircv3.net/specs/extensions/sasl-3.1)
- [Atheme IRC services](https://github.com/atheme/atheme)
