---
order: 3
title: "Injection d'export"
description: "Des données contrôlées par l'attaquant stockées par une application web sont ensuite interprétées par un programme en aval qui ouvre le fichier exporté, déplaçant l'exécution de code hors du serveur web."
keywords:
  - injection d'export
  - injection csv
  - injection de formule
  - injection latex
  - html vers pdf
---

# Exports

L'injection d'export inverse le sens de l'attaque : le payload est stocké comme donnée ordinaire, et le dommage survient lorsqu'un **programme en aval** ouvre le fichier que l'application exporte.

Un attaquant écrit une valeur dans un champ (nom, note, adresse) que l'application émet plus tard dans un artefact généré. Le fichier exporté est ensuite analysé par un second interpréteur qui traite une partie des données comme des instructions : un tableur évaluant une cellule qui commence par `=`, un moteur TeX honorant une macro, ou un rendu HTML-vers-PDF récupérant et exécutant du balisage injecté. L'application web peut être parfaitement sûre ; l'exécution atterrit dans le client tableur de la victime, dans la chaîne d'outils PDF côté serveur, ou contre des endpoints internes que le moteur de rendu peut atteindre. Les trois pages couvrent les formules CSV/tableur, les pipelines LaTeX et les moteurs de rendu HTML-vers-PDF.

## Pages

- **[CSV](csv.md)** : Les cellules exportées commençant par =, +, - ou @ sont évaluées comme des formules par les tableurs, permettant l'exécution de commande via DDE et l'exfiltration de données via l...
- **[LaTeX](latex.md)** : Les champs contrôlés par l'attaquant compilés par un moteur TeX côté serveur permettent la lecture de fichiers via \\input et l'exécution de commandes shell via \\write18 lorsque shell-escape est a...
- **[PDF](pdf.md)** : Le HTML et le JavaScript injectés dans les moteurs de rendu PDF côté serveur lisent des fichiers locaux via file://, atteignent des endpoints internes et de métadonnées cloud via SSRF, et exfiltr...

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour soumettre des payloads dans les champs exportés.
- **LibreOffice / application bureautique** : ouvrir les fichiers générés pour vérifier l'exécution en aval.

## Références

- [OWASP: CSV Injection](https://owasp.org/www-community/attacks/CSV_Injection)
- [PayloadsAllTheThings: CSV Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CSV%20Injection)
