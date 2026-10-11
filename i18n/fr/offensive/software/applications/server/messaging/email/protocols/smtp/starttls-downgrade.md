---
title: "Downgrade STARTTLS : dépouiller et injecter autour du TLS opportuniste"
description: "STARTTLS sur le port 25 est opportuniste, un attaquant bien positionné peut donc réécrire la liste de capacités EHLO pour supprimer STARTTLS et maintenir le pair en clair, là où AUTH et MAIL sont interceptables. Séparément, la classe d'injection de commandes avant TLS exécute comme des commandes les octets mis en tampon avant le handshake, à l'intérieur de la session chiffrée."
keywords:
  - starttls
  - tls downgrade
  - stripping
  - command injection
  - smtp mitm
---

# Downgrade STARTTLS

STARTTLS met à niveau une connexion SMTP en clair vers TLS sur place, mais sur le port 25 la mise à niveau est opportuniste : le client ne la tente que si le serveur annonce `250-STARTTLS` dans sa réponse EHLO, et si elle manque le client poursuit en clair. Ce « si annoncé » est la faiblesse. Deux attaques distinctes y vivent. La première est le dépouillement : une machine en position d'intermédiaire réécrit la réponse EHLO du serveur pour supprimer la capacité STARTTLS, de sorte que le pair ne se met jamais à niveau et envoie AUTH et MAIL en clair. La seconde est l'injection de commandes avant TLS : certains MTA et proxys terminant TLS n'écartent pas le tampon de réception en clair quand le handshake commence, de sorte que les commandes qu'un attaquant ajoute après `STARTTLS` (envoyées avant TLS) sont analysées comme si elles étaient arrivées à l'intérieur de la session chiffrée désormais de confiance.

## Prérequis

- Le dépouillement nécessite une position sur le chemin entre les deux pairs SMTP (redirection ARP/DNS sur le LAN, un amont malveillant, un saut de relais contrôlé). Il n'aide que là où TLS n'est pas obligatoire, ce qui est le cas par défaut pour le port 25 entrant.
- Confirmez que STARTTLS est réellement proposé et joignable avant de l'attaquer :

```bash
# does the server advertise and complete STARTTLS?
openssl s_client -starttls smtp -connect mail.victim.com:25 -crlf
# ... reaches a TLS handshake and cert => STARTTLS works and is a downgrade target
echo | openssl s_client -starttls smtp -connect mail.victim.com:25 2>/dev/null | openssl x509 -noout -subject
```
Un handshake abouti confirme que le TLS opportuniste est en jeu ; un serveur en clair uniquement n'a rien à dépouiller mais est déjà interceptable.

## Dépouiller la capacité

L'intermédiaire transmet le dialogue mais édite l'unique ligne qui propose la mise à niveau, de sorte que le test « STARTTLS est-il disponible ? » du client échoue :

```text
# server actually sends:
250-mail.victim.com
250-STARTTLS            <-- attacker deletes or mangles this line in transit
250 AUTH LOGIN PLAIN
# client sees no STARTTLS, stays in cleartext, and sends:
AUTH PLAIN AGpzbWl0aAB...   <-- credential captured on the wire
```
La dégradation (réécrire `250-STARTTLS` en `250-XXXXXXXX`, ou la commande `STARTTLS` du client en `XXXXXXXX`, de sorte que le verbe soit méconnu et reçoive un `500`) obtient le même résultat sans changer la longueur de ligne. Après le dépouillement, lisez l'`AUTH`/`MAIL FROM` en clair exactement comme dans [authentification](authentication.md).

## Injection de commandes avant TLS

Ici vous ne dépouillez pas TLS ; vous faites passer clandestinement une commande à travers la frontière du handshake. Tout ce qui suit `STARTTLS\r\n` dans le même segment TCP est du texte en clair que le serveur devrait jeter, mais une implémentation vulnérable le conserve et l'exécute après le handshake, à l'intérieur du contexte authentifié/chiffré :

```text
EHLO attacker.test\r\n
STARTTLS\r\nRSET\r\n        <-- "RSET" (or MAIL FROM/AUTH) is buffered BEFORE TLS
<TLS handshake begins>
                           <-- after TLS, the buffered RSET runs as a trusted command
```
Remplacez `RSET` par une commande significative : injecter `MAIL FROM`/`RCPT TO` permet à un attaquant qui présente ensuite un certificat client ou profite d'une session de confiance d'ajouter en préfixe une transaction que le serveur attribue au canal sécurisé. Cette faille de mise en tampon a affecté plusieurs MTA et proxys de messagerie.

## Suites

- Après un dépouillement, vous disposez des identifiants et du contenu des messages en clair : injectez les identifiants dans l'accès à la boîte aux lettres et dans une réutilisation plus large.
- Avec l'injection avant TLS, vous pouvez injecter des commandes dans la session que le serveur considère de confiance ; gardez ceci conceptuellement distinct du [SMTP smuggling](smtp-smuggling.md), qui abuse de l'analyse de la fin de données entre deux serveurs plutôt que de la frontière TLS sur une connexion.

## Outils

- [openssl s_client](https://www.openssl.org/docs/man3.0/man1/openssl-s_client.html)
- [Ettercap](https://github.com/Ettercap/ettercap)

## Références

- [RFC 3207 (SMTP over TLS, STARTTLS)](https://datatracker.ietf.org/doc/html/rfc3207)
- [RFC 3207 section 4.2 (écarter les données en tampon après STARTTLS)](https://datatracker.ietf.org/doc/html/rfc3207#section-4.2)
- [Scripts smtp-vuln de Nmap](https://nmap.org/nsedoc/categories/vuln.html)
