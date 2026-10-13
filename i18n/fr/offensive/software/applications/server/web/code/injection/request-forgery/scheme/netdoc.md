---
title: "Schéma Netdoc pour la lecture de fichier local sur la JVM"
description: "netdoc: est un gestionnaire d'URL Java hérité qui lit des fichiers locaux, et sert donc d'alternative à file:// lorsque ce schéma est filtré sur une stack JVM."
keywords:
  - schéma netdoc
  - netdoc://
  - lecture de fichier Java
  - alternative au schéma file
  - divulgation de fichier local
---

# Netdoc

`netdoc:` est un ancien gestionnaire d'URL Java qui se résout en une lecture de fichier local. Il est antérieur à `file:` et lui est parallèle, et parce que les défenseurs mettent souvent `file://` sur liste de blocage en oubliant `netdoc:`, c'est une alternative utile pour lire des fichiers locaux sur une stack JVM.

## Lire un fichier

Le gestionnaire prend un chemin à la manière de `file:` :

```
netdoc:/etc/passwd
netdoc:///etc/passwd
netdoc:/proc/self/environ
```

Sur un client Java qui expose encore le gestionnaire, ceci renvoie le contenu du fichier via le même canal que celui qui a reflété la récupération, atteignant les mêmes cibles que le schéma [File](file.md) : la configuration avec identifiants, l'environnement, les sources et les clés.

## Pourquoi il existe comme page distincte

La seule raison d'utiliser `netdoc:` plutôt que `file:` est l'évasion. Une liste d'autorisation ou de blocage de schémas écrite pour `http`/`https`/`file` omet couramment le gestionnaire hérité, de sorte que lorsque `file://` est rejeté mais que la stack est Java, `netdoc:` atteint le système de fichiers tout de même. Il a sa place dans la rotation aux côtés de [File](file.md) et [JAR](jar.md) : lorsqu'un schéma local accessible en Java est filtré, essayez les autres avant de conclure que la lecture locale est indisponible.

## Outils

- **Client URL Java** : `new URL("netdoc:/etc/passwd").openStream()` exerce directement ce gestionnaire propre à la JVM, puisque curl ne prend pas en charge le schéma `netdoc:`.
- **Burp Repeater** : soumettez l'URL `netdoc:` via le sink de récupération JVM de la cible et lisez le contenu reflété.

## Références

- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
