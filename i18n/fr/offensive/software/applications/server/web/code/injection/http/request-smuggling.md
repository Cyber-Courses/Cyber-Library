---
order: 5
title: "HTTP request smuggling (désync)"
description: "Désynchroniser un front-end et un back-end qui mesurent la longueur de requête différemment, en utilisant Content-Length et Transfer-Encoding dans les variantes CL.TE, TE.CL et TE.TE."
keywords:
  - HTTP request smuggling
  - désync
  - CL.TE
  - TE.CL
  - TE.TE
  - Transfer-Encoding
---

# Request smuggling

Le request smuggling (désync) abuse d'une chaîne où un proxy front-end et un serveur back-end divergent sur l'endroit où une requête se termine et où la suivante commence. HTTP/1.1 offre deux façons d'indiquer la longueur d'un corps, `Content-Length` (CL) et `Transfer-Encoding: chunked` (TE) ; si les deux serveurs font confiance à l'un des deux différemment, un attaquant peut cacher le début d'une seconde requête à l'intérieur de la première. Ce préfixe passé en fraude est alors collé à la requête du client suivant, permettant le contournement du contrôle front-end, la capture de requêtes, et l'empoisonnement de cache.

**CL.TE** : le front-end utilise `Content-Length`, le back-end utilise `Transfer-Encoding`. Le front transmet tout le corps par comptage d'octets ; le back s'arrête au terminateur chunked `0\r\n\r\n`, laissant les octets de fin comme début de la requête suivante :

```
POST / HTTP/1.1
Host: victim
Content-Length: 6
Transfer-Encoding: chunked

0

G
```

Le back-end traite `0\r\n\r\n` comme la fin et laisse `G`, qui préfixe la requête suivante (la transformant en `GPOST ...` ou en une requête passée en fraude forgée).

**TE.CL** : l'inverse, le front-end honore `Transfer-Encoding`, le back-end honore `Content-Length`. Les tailles de chunk sont forgées de sorte que le `Content-Length` du back-end coupe le corps tôt, faisant passer le reste en fraude.

**TE.TE** : les deux supportent `Transfer-Encoding`, mais l'un est induit à l'ignorer par un en-tête obfusqué qu'un parseur accepte et l'autre rejette (`Transfer-Encoding: xchunked`, un espace avant les deux-points, un en-tête dupliqué, ou `Transfer-Encoding:\tchunked`). Quel que soit le bout qui retombe sur `Content-Length`, il désynchronise alors comme CL.TE ou TE.CL.

Les payloads sont sensibles aux octets (CR/LF exacts, tailles de chunk), et le smuggling par downgrade HTTP/2 étend l'idée là où un front-end réécrit h2 en h1. Confirmez un désync avec du timing ou un préfixe bénin avant de l'armer, car une longueur erronée peut casser la connexion partagée.

## Outils

- **Burp HTTP Request Smuggler** : extension Burp pour la détection et l'exploitation automatisées du désync de requêtes.
- **Turbo Intruder** : envoi de requêtes à haut volume et en paquet unique pour le smuggling et les sondes de timing.
- **smuggler.py** : scanner en ligne de commande pour les variantes de désync CL.TE, TE.CL et TE.TE.
- **Burp Repeater** : forger manuellement des payloads CL/TE sensibles aux octets et confirmer le désync.

## Références

- PortSwigger Web Security Academy: HTTP request smuggling
- RFC 9112: HTTP/1.1 message length (Content-Length vs Transfer-Encoding)
