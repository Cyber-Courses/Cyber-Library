---
title: "Sélection et scan de ports à travers un sink SSRF"
description: "Faire varier le port dans une URL fetchable transforme un SSRF aveugle en scanner de ports interne, utilisant le timing de réponse et les différences d'erreur pour cartographier quels services écoutent."
keywords:
  - scan de ports SSRF
  - scan de ports interne
  - oracle de timing de réponse
  - connection refused
  - découverte de services
---

# Port

Le port dans l'authority décide quel service sur l'hôte répond. En l'itérant contre une adresse interne, un sink de fetch devient un scanner de ports qui cartographie le réseau interne depuis le point de vue du serveur, même lorsqu'aucun corps de réponse n'est renvoyé.

## L'oracle de timing et d'erreur

Un sink aveugle ne renvoie aucun contenu, mais la *manière* dont il échoue fuit encore l'état du port. Trois issues se distinguent généralement nettement :

- **Ouvert** : la connexion TCP réussit. Le service peut parler un protocole que le client ne comprend pas, donc la requête reste suspendue jusqu'à un read timeout, ou renvoie une erreur de protocole ou de parse après connexion. Dans tous les cas la phase de connexion est rapide.
- **Fermé** : l'hôte envoie un RST immédiat, donc le client rapporte connection refused presque instantanément.
- **Filtré** : un pare-feu laisse tomber le paquet, donc le client bloque jusqu'à un connect timeout complet, nettement plus long que les deux autres.

Mesurer le temps de réponse par port, et lire toute chaîne d'erreur que le sink reflète, classe chaque port sans aucune réponse applicative.

```
http://10.0.0.5:22/      # fast connect, then banner or protocol error  -> open
http://10.0.0.5:3306/    # fast connect                                 -> open
http://10.0.0.5:8081/    # immediate refused                            -> closed
http://10.0.0.5:9999/    # long hang to timeout                         -> filtered
```

## Quels ports balayer

Scanner les 65535 ports à travers un sink lent vaut rarement la peine ; ciblez les services qui comptent :

```
22 80 443 2375 3306 5432 6379 8080 8443 9200 11211 27017
```

Ceux-ci couvrent SSH, les frontaux web, l'API Docker, les bases de données courantes, Redis, Elasticsearch, memcached et MongoDB. En trouver un ouvert restreint ce qu'il faut attaquer ensuite, et le payload de scheme correspondant (par exemple [Gopher](../scheme/gopher.md) sur `6379`) interagit alors avec lui.

## Loopback versus voisins

Pointez le scan vers `127.0.0.1` pour trouver les services liés uniquement à la loopback (panneaux d'administration, serveurs de debug, bases de données écoutant localement), invisibles depuis l'extérieur de l'hôte mais atteignables à travers le serveur lui-même. Puis pivotez vers les voisins RFC1918 (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) pour cartographier les hôtes adjacents. Combinez avec les encodages d'[adresse IP](ip-address.md) lorsque l'adresse littérale est filtrée.

## Outils

- **SSRFmap** : scan de ports interne automatisé et découverte de services à travers un sink SSRF.
- **curl** : itérer une liste de ports contre un hôte interne et lire les différences de timing ou d'erreur à la main.
- **nmap** : construire une liste de ports de services courants à injecter dans le sink (par exemple `22,80,443,3306,6379,8080`).

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
