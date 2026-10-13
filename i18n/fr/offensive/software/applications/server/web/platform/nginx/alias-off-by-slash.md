---
order: 1
title: "Off-by-slash d'alias nginx : traversée vers le répertoire parent"
description: "Exploitation du off-by-slash location/alias classique de nginx : un préfixe de location sans slash final et un alias avec en concatènent les caractères suivants, permettant à ../ d'atteindre des fichiers hors du répertoire mappé."
keywords:
  - traversée alias nginx
  - off by slash
  - location alias
  - traversée de chemin
---

# Off-by-slash d'alias

Le bug de service de fichiers nginx le plus courant : un préfixe `location` sans slash final combiné à un `alias` qui en a un. nginx retire le texte du `location` correspondant et ajoute le reste de l'URI à l'`alias`, de sorte que les caractères immédiatement après le préfixe, `..` compris, se concatènent directement sur le chemin du système de fichiers.

## La mauvaise configuration

```nginx
location /static {
    alias /var/www/app/static/;
}
```

Le `location` est `/static` (pas de slash final). Une requête pour `/static../config.php` a `../config.php` restant après le retrait du préfixe, ajouté à l'alias :

```
/var/www/app/static/ + ../config.php  ->  /var/www/app/config.php
```

Donc :

```
GET /static../app.py HTTP/1.1
GET /static../../etc/passwd HTTP/1.1
GET /static../.env HTTP/1.1
```

La forme correcte fait correspondre `location /static/` avec le slash final (alors `/static..` ne correspond pas du tout au location), mais la forme vulnérable est partout.

## La trouver

- Notez les préfixes statiques depuis les URL d'assets dans le HTML/JS (`/static`, `/assets`, `/media`, `/files`, `/images`).
- Pour chacun, demandez `<préfixe>../` plus une cible : d'abord le répertoire parent (`../config.php`, `../.env`, `../app.py`, `../wp-config.php`), puis remontez (`../../etc/passwd`).
- Essayez la traversée brute et encodée (`..%2f`, `%2e%2e/`) au cas où un filtre se trouve devant.
- Utilisez `curl --path-as-is` pour que le client ne réduise pas les `..` avant l'envoi.

```bash
curl --path-as-is https://target/static../config.php
```

## Exploitation

- Un niveau au-dessus d'un répertoire statique contient presque toujours le code source et la configuration de l'application ; récupérez-les en premier (ils alimentent toutes les autres attaques).
- Le `.env`/config récupéré livre clés secrètes et identifiants ; le code source récupéré révèle les sinks d'injection et de désérialisation.
- Remontez vers `/etc/passwd`, la config nginx (`/etc/nginx/nginx.conf`) et les logs applicatifs pour un levier supplémentaire.

## Outils

- **curl --path-as-is**, Burp Intruder avec une wordlist de traversée ; templates **nuclei** nginx-alias.

## Références

- Documentation nginx : location, alias
- Orange Tsai : recherche nginx off-by-slash
