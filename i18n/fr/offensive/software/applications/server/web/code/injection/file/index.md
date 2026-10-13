---
order: 12
title: "Injection de fichiers"
description: "Abus de la façon dont une application web reçoit, résout, inclut ou exporte des fichiers, transformant la manipulation de fichiers en exécution de code, divulgation, ou attaques sur un interpréteur en aval."
keywords:
  - injection de fichiers
  - upload de fichiers
  - inclusion de fichiers locale
  - inclusion de fichiers distante
  - injection d'export
---

# File

L'injection de fichiers regroupe les attaques qui abusent de la façon dont une application web reçoit, résout et émet des fichiers, plutôt que de la façon dont elle construit une requête.

Trois axes parcourent ce sous-arbre. L'**upload** couvre l'écriture d'octets contrôlés par l'attaquant sur un chemin que le serveur exécutera ou servira ensuite, y compris l'implantation de web shell et l'extraction d'archive qui s'échappe de son répertoire cible. L'**inclusion** couvre les sinks de type `include`/`require` dynamiques où un chemin ou un fragment d'URL est contrôlé par l'attaquant, aboutissant à la divulgation de source, à la lecture de fichiers locaux et à l'exécution de code. Les **exports** couvrent la direction inverse : des données que l'application écrit (CSV, LaTeX, PDF) qu'un programme en aval analyse et interprète, transformant une fonctionnalité d'export en exécution de commandes ou en vol de données sur un autre système. Chaque page de technique porte des payloads concrets et exécutables.

## Sous-sujets

- **[Exports](exports/index.md)** : des données contrôlées par l'attaquant stockées par une application web sont ensuite interprétées par un programme en aval qui ouvre le fichier exporté, déplaçant l'exécution de code hors du serveur we...
- **[Inclusion](inclusion/index.md)** : sinks include et require dynamiques où un chemin ou un fragment d'URL contrôlé par l'attaquant choisit ce que l'interpréteur charge, aboutissant à une divulgation ou à l'exécution de code.
- **[Upload](upload/index.md)** : transformer une fonctionnalité d'upload en exécution de code ou en écrasement de fichier en venant à bout des contrôles sur le nom, le type, le contenu et la destination des fichiers fournis par l'attaquant.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater et Intruder pour les payloads d'upload, d'inclusion et d'export.
- **[fimap](https://github.com/kurobeats/fimap)** : test automatisé d'inclusion de fichiers.
- **[exiftool](https://exiftool.org/)** : intégrer des payloads dans les métadonnées de fichiers pour des polyglottes.

## Références

- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
