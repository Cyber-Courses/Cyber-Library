---
title: "Divulgation d'environnement SSI"
description: "La directive echo affiche les variables d'environnement SSI et CGI dans la page, fuitant des chemins, des données client et la configuration serveur lorsque l'entrée atteint un document interprété en SSI."
keywords:
  - SSI echo var
  - divulgation d'environnement
  - DOCUMENT_NAME
  - HTTP_USER_AGENT
  - variables CGI
---

# Divulgation d'environnement

La directive `echo` affiche la valeur d'une variable d'environnement SSI ou CGI dans la page. Là où `exec` est bloqué, `echo` est souvent toujours activé, et c'est un moyen fiable de confirmer que le SSI est évalué et de récupérer l'état du serveur dans la réponse sans exécuter de commande.

## La directive

```
<!--#echo var="DOCUMENT_NAME"-->
<!--#echo var="DATE_LOCAL"-->
```

Une valeur reflétée prouve que la directive a été interprétée. `DOCUMENT_NAME` et `DATE_LOCAL` sont des variables SSI standard et constituent des sondes propres et peu risquées.

## Variables intégrées

`mod_include` expose un ensemble fixe de variables décrivant le document et la requête en cours :

```
<!--#echo var="DOCUMENT_NAME"-->
<!--#echo var="DOCUMENT_URI"-->
<!--#echo var="DATE_LOCAL"-->
<!--#echo var="DATE_GMT"-->
<!--#echo var="LAST_MODIFIED"-->
```

`DOCUMENT_URI` est le chemin d'URL de la requête et `DOCUMENT_NAME` le nom du document demandé, pas le vrai chemin sur disque, donc ils confirment où l'entrée reflétée est rendue dans l'espace d'URL. Le chemin absolu sur le système de fichiers provient de `SCRIPT_FILENAME` ou `PATH_TRANSLATED` ci-dessous, quand le serveur les expose.

## Variables CGI et de requête

L'ensemble des variables CGI divulgue bien davantage. Les en-têtes contrôlés par le client et les métadonnées de connexion sont tous accessibles :

```
<!--#echo var="HTTP_USER_AGENT"-->
<!--#echo var="HTTP_COOKIE"-->
<!--#echo var="REMOTE_ADDR"-->
<!--#echo var="REMOTE_PORT"-->
<!--#echo var="SERVER_SOFTWARE"-->
<!--#echo var="SERVER_NAME"-->
<!--#echo var="SERVER_ADDR"-->
<!--#echo var="QUERY_STRING"-->
<!--#echo var="PATH_TRANSLATED"-->
<!--#echo var="SCRIPT_FILENAME"-->
```

`SERVER_SOFTWARE` identifie la pile. `SCRIPT_FILENAME` et `PATH_TRANSLATED` révèlent des chemins absolus du système de fichiers, qui alimentent directement les charges utiles d'inclusion de fichier et de commande nécessitant un chemin exact. `HTTP_COOKIE` peut exposer la valeur de session d'un autre utilisateur quand la directive est stockée et rendue dans une vue d'administration.

## Reflet de variables contrôlées par l'attaquant

Parce que les variables dérivées des en-têtes telles que `HTTP_USER_AGENT` renvoient l'entrée de la requête, elles servent aussi de recoupement. Placez un marqueur dans l'en-tête et relisez-le via `echo` :

```
User-Agent: SSIPROBE-7f3a
```

```
<!--#echo var="HTTP_USER_AGENT"-->
```

Voir `SSIPROBE-7f3a` dans la réponse confirme à la fois que le SSI s'exécute et que l'en-tête atteint la page interprétée.

## Variables personnalisées et configurées

Les variables définies avec `set`, ou les variables CGI exportées par l'environnement de l'application, sont lisibles par leur nom. Énumérez les noms probables une fois le logiciel serveur connu :

```
<!--#set var="probe" value="x"--><!--#echo var="probe"-->
<!--#echo var="HTTPS"-->
<!--#echo var="SERVER_PROTOCOL"-->
```

Une variable inexistante s'affiche généralement comme `(none)`, ce qui distingue en soi une directive interprétée d'une directive qui a été renvoyée telle quelle.

## Formater la sortie

La directive `config` contrôle la façon dont `echo` rend les valeurs, et `timefmt` en particulier change le format des variables de date. La définir avant un `echo` est un moyen propre et sans effet de bord de prouver que le parseur est actif et de faire varier la sortie pour que la mise en cache ou le filtrage ne masque pas la réflexion :

```
<!--#config timefmt="%Y-%m-%d %H:%M:%S"--><!--#echo var="DATE_LOCAL"-->
```

`config errmsg` remplace la chaîne que le serveur émet quand une directive ultérieure échoue, ce qui est utile pour distinguer une directive interprétée-mais-échouée d'une directive qui n'a jamais été évaluée :

```
<!--#config errmsg="SSI-ACTIVE"--><!--#include virtual="/does-not-exist"-->
```

Si la réponse contient `SSI-ACTIVE` à la place du texte d'erreur par défaut, le flux de directives est bien interprété même si l'inclusion elle-même a échoué.

## Enchaîner la divulgation vers d'autres primitives

La divulgation d'environnement est généralement une étape intermédiaire. `SCRIPT_FILENAME` et `PATH_TRANSLATED` donnent l'emplacement absolu sur disque de la page interprétée (tandis que `DOCUMENT_URI` ne donne que son chemin d'URL), ce qui permet aux charges utiles d'inclusion de fichier et de commande d'utiliser des chemins exacts plutôt que d'empiler des séquences de traversée à l'aveugle. `SERVER_SOFTWARE` et `SERVER_PROTOCOL` identifient la pile pour que les charges utiles ultérieures correspondent au serveur. Collectez-les d'abord, puis pivotez vers `exec` ou `include` avec les chemins et détails de version déjà en main.

## Outils

- **Burp Repeater** : envoi de sondes `echo var` et lecture des valeurs reflétées.
- Tests manuels avec des directives `<!--#echo var-->` et `<!--#config-->` forgées.

## Références

- [OWASP: Server-Side Includes (SSI) Injection](https://owasp.org/www-community/attacks/Server-Side_Includes_(SSI)_Injection)
- [Apache mod_include documentation](https://httpd.apache.org/docs/current/mod/mod_include.html)
