---
title: "Scheme File pour la lecture de fichiers locaux via SSRF"
description: "Le handler file:// fait lire à un client d'URL un chemin local au lieu d'une ressource réseau, renvoyant le contenu des fichiers que le compte de service peut ouvrir."
keywords:
  - scheme file
  - file://
  - lecture de fichier local
  - /etc/passwd
  - divulgation de fichier SSRF
---

# File

Lorsqu'un client d'URL honore `file://`, pointer le sink vers un chemin local lui fait lire un fichier depuis le disque du serveur lui-même et renvoyer le contenu à travers le canal qui reflétait le fetch. C'est l'escalade de scheme la plus simple : une fonctionnalité censée récupérer une URL distante devient un lecteur de fichiers locaux.

## Lire un fichier

Le chemin suit le scheme, avec une authority vide ou `localhost` :

```
file:///etc/passwd
file://localhost/etc/passwd
file:///proc/self/environ
file:///proc/self/cwd/app.py
file:///home/app/.ssh/id_rsa
```

Sur des cibles Windows, le chemin utilise une lettre de lecteur :

```
file:///C:/Windows/win.ini
file:///C:/inetpub/wwwroot/web.config
```

Tout fichier que le compte de service peut ouvrir est atteignable : la configuration avec des credentials embarqués, l'environnement via `/proc/self/environ`, le code source et les clés privées sont les objectifs habituels.

## Lorsque la réponse est reflétée

`file://` est le plus utile lorsque le contenu récupéré revient, par exemple un aperçu, un import qui renvoie en écho ce qu'il a lu, ou une erreur qui inclut le corps. Lorsque le contenu n'est pas reflété, une lecture aveugle confirme quand même que le scheme est honoré (via le timing ou une erreur qui diffère pour un chemin existant par rapport à un chemin manquant), ce qui vaut la peine d'être connu car cela implique que d'autres schemes locaux tels que [Netdoc](netdoc.md) peuvent aussi fonctionner.

## Interaction scheme et parseur

Atteindre `file://` requiert souvent de franchir une allowlist de schemes. Un client qui valide le scheme soumis mais suit une [redirection](../query/bypassing-using-a-redirect.md) dont le `Location` est `file:///etc/passwd`, ou un parseur d'URL qui lit mal le scheme, réussit la lecture même lorsque `file://` est nominalement bloqué. Les clients Java exposent `file:` aux côtés de [JAR](jar.md) et [Netdoc](netdoc.md), donc si l'un est filtré les autres valent la peine d'être essayés.

## Outils

- **SSRFmap** : exploitation SSRF automatisée avec un module pour la lecture locale `file://`.
- **curl** : sondage manuel `file://` pour lire un chemin local et inspecter le contenu reflété.
- **Burp Repeater** : pour les lectures aveugles, utiliser l'oracle de timing ou d'erreur chemin-existant-versus-manquant ; une lecture du système de fichiers local ne produit aucune interaction out-of-band, donc un serveur d'interaction ne peut pas la confirmer.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
