---
order: 2
title: "Inclusion de fichiers"
description: "Sinks include et require dynamiques où un chemin ou un fragment d'URL contrôlé par l'attaquant choisit ce que l'interpréteur charge, aboutissant à une divulgation ou à l'exécution de code."
keywords:
  - inclusion de fichiers
  - LFI
  - RFI
  - include
  - require
  - wrappers php
---

# Inclusion

Les failles d'inclusion vivent dans des sinks de type `include`/`require` dynamiques, où un paramètre de requête décide quel fichier l'interpréteur charge et exécute.

Du code PHP classique comme `include($_GET['page'] . '.php')` confie à l'attaquant le contrôle du chemin chargé. Deux directions s'ensuivent. L'inclusion **locale** pointe le sink vers des fichiers déjà présents sur le serveur : traversée pour lire le source et les secrets, wrappers pour divulguer ou décoder, et astuces d'empoisonnement (logs, `/proc`, fichiers de session, uploads antérieurs) qui convertissent une primitive de lecture en exécution de code. L'inclusion **distante** pointe le sink vers une URL hébergée par l'attaquant, de sorte que l'interpréteur récupère et exécute directement du code externe, sous réserve des paramètres d'inclusion d'URL du moteur. Les deux pages couvrent les payloads et les chaînes qui promeuvent une simple lecture de fichier en exécution complète.

## Pages

- **[Local](local.md)** : LFI par traversée et wrappers PHP pour la divulgation de source, plus des chaînes d'empoisonnement de log, environ, session et upload qui escaladent une lecture de fichier en exécution de co...
- **[Remote](remote.md)** : le RFI pointe un sink d'inclusion vers une URL contrôlée par l'attaquant de sorte que l'interpréteur récupère et exécute du code externe, conditionné par allow_url_include et allow_url_fopen.

## Outils

- **[LFISuite](https://github.com/D35m0nd142/LFISuite)** : détection et exploitation automatisées de LFI.
- **[kadimus](https://github.com/P0cL4bs/Kadimus)** : scan et exploitation de LFI et RFI.
- **[fimap](https://github.com/kurobeats/fimap)** : test automatisé d'inclusion de fichiers locale et distante.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour les payloads de traversée, de wrapper et d'inclusion d'URL.

## Références

- [OWASP: Testing for Local File Inclusion](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)
