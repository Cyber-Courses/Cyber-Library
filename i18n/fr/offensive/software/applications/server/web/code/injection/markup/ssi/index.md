---
title: "Injection SSI"
description: "Les Server-Side Includes étendent les directives <!--#...--> avant qu'une page ne soit servie ; l'entrée utilisateur qui atteint une page interprétée en SSI exécute des directives permettant l'exécution de commandes, la divulgation de variables et l'inclusion de fichiers."
keywords:
  - injection SSI
  - Server-Side Includes
  - injection shtml
  - exec cmd
  - include virtual
---

# SSI

Les Server-Side Includes sont des directives intégrées au HTML que le serveur web étend au moment de la réponse, avant que la page n'atteigne le client. Elles prennent la forme `<!--#directive parameter="value"-->`, qui ressemble à un commentaire HTML mais est interprétée et exécutée par le serveur. Apache (`mod_include`), nginx (`ssi on`) et IIS supportent tous ce mécanisme, classiquement sur les fichiers avec une extension `.shtml`, `.shtm` ou `.stm`, bien que n'importe quel type de contenu puisse être configuré pour le parsing SSI.

L'injection se produit lorsqu'une entrée contrôlée par l'attaquant est écrite dans une page que le serveur interprète ensuite pour y chercher des directives SSI. Si un champ de commentaire, un nom de fichier, une ligne de log ou toute valeur reflétée atterrit dans un document avec le SSI activé et que le serveur l'évalue, une directive `<!--#...-->` injectée s'exécute avec les privilèges du processus du serveur web. L'attaquant n'a pas besoin de contrôler le fichier sur disque ; l'entrée doit seulement être présente dans le flux d'octets au moment où le parseur SSI s'exécute.

La portée pratique dépend des directives que le serveur autorise. `exec` atteint les commandes du système d'exploitation et est la primitive à plus fort impact. `echo` fuite l'état du serveur via les variables d'environnement intégrées et CGI. `include` lit des fichiers et intègre des ressources côté serveur, la traversée de chemin étendant sa portée. `config`, `set` et `fsize` complètent l'ensemble des directives et soutiennent l'identification et l'enchaînement.

Ce sous-arbre couvre l'exécution de commandes via `exec`, la divulgation d'environnement via `echo` et l'inclusion de fichier via `include`.

## Pages

- **[Exécution de commandes](command-execution.md)** : la directive exec exécute des commandes du système d'exploitation lorsqu'une entrée attaquant atteint une page interprétée en SSI, transformant du balisage reflété en exécution de code à distance sous le...
- **[Divulgation d'environnement](environment-disclosure.md)** : la directive echo affiche les variables d'environnement SSI et CGI dans la page, fuitant des chemins, des données client et la configuration serveur lorsque l'entrée atteint un document interprété en SSI...
- **[Inclusion de fichier](file-inclusion.md)** : la directive include intègre des fichiers et des ressources côté serveur dans une page interprétée en SSI ; les chemins file et virtual ainsi que la traversée permettent à un attaquant de lire des fichiers locaux...

## Outils

- **Burp Suite** : réflexion et itération de directives SSI injectées via Repeater et Intruder.
- **Burp Collaborator** : confirmation de l'exécution aveugle de directives hors bande.
- Tests manuels avec des directives `<!--#...-->` forgées.

## Références

- OWASP : Server-Side Includes (SSI) Injection
- OWASP WSTG : Testing for Server-Side Includes (SSI) Injection
