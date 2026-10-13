---
title: "Scheme DICT pour l'interaction avec des services via SSRF"
description: "Le handler dict:// ouvre une connexion TCP et envoie une seule ligne de commande, ce qui sonde les bannières et pilote des services internes orientés ligne tels que Redis et memcached."
keywords:
  - scheme dict
  - dict://
  - SSRF Redis
  - memcached
  - bannière de service
  - protocole orienté ligne
---

# DICT

Le scheme `dict://`, supporté par les clients adossés à curl, ouvre une connexion TCP vers l'hôte et le port de l'URL et envoie une ligne de commande construite à partir du chemin. Il a été conçu pour interroger des serveurs de dictionnaires, mais parce qu'il écrit une ligne choisie par l'attaquant vers un port arbitraire et renvoie la réponse, il fait aussi office de moyen de parler à tout service interne orienté ligne.

## Forme de la requête

La forme canonique transporte une commande et des arguments après l'hôte :

```
dict://127.0.0.1:6379/info
dict://127.0.0.1:11211/stats
```

Le client se connecte au port et envoie le mot comme une ligne, puis lit la réponse. curl préfixe sa propre ligne d'annonce `CLIENT` avant la commande, donc un service orienté ligne voit une ligne supplémentaire en premier ; Redis et memcached la tolèrent et traitent quand même la commande qui suit. Redis répond à `info` par son bloc serveur, et memcached répond à `stats`, les deux confirmant le service et faisant fuiter la version et la configuration.

## Une commande par requête

`dict://` envoie une seule ligne par requête, il est donc idéal pour le sondage et les commandes ponctuelles plutôt que pour des séquences multi-étapes. Une commande avec arguments doit encoder en pourcentage ses espaces, car un client adossé à curl rejette une URL contenant un espace littéral avant de se connecter :

```
dict://127.0.0.1:11211/stats%20items     # memcached: stats items
dict://127.0.0.1:6379/COMMAND%20DOCS      # Redis: one command, spaces encoded
```

Chaque requête est une connexion fraîche et une seule commande. Lire une bannière de service ou une réponse à une seule commande de cette façon identifie ce qui écoute sur un port découvert via un scan de [Port](../authority/port.md). Lorsqu'une séquence de commandes est nécessaire (par exemple la chaîne complète d'écriture-sur-disque de Redis, ou toute valeur contenant des sauts de ligne), la limite d'une seule ligne est la raison de passer à [Gopher](gopher.md), qui écrit tout un flux confectionné en une seule requête.

## Sonder fermé versus ouvert

Parce que la réponse (ou l'erreur de connexion) revient, `dict://` distingue un port ouvert qui parle un protocole connu d'un qui ne le parle pas, affinant un scan de ports en identification de services. Pointez-le vers chaque port candidat et lisez si une bannière reconnaissable revient.

## Outils

- **SSRFmap** : exploitation SSRF automatisée contre les cibles internes courantes telles que Redis et memcached.
- **curl** : sondage manuel `dict://` d'un port et lecture de la réponse d'une seule ligne.
- **Burp Collaborator** : détection d'interaction out-of-band lorsque la réponse du service n'est pas reflétée.

## Références

- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [curl: DICT protocol](https://curl.se/docs/url-syntax.html)
