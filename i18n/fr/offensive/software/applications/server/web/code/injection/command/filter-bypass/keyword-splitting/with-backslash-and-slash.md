---
order: 4
title: "Fragmentation de mots-clés avec antislash et slash : w\\ho\\am\\i et /bin/c\\at"
description: "Fragmenter un mot-clé bloqué avec des échappements par antislash, w\\ho\\am\\i, et insérer des slashs redondants dans les chemins, /bin/c\\at, //bin//cat, pour que le shell normalise le token et qu'une blocklist littérale le manque."
keywords:
  - injection de commande
  - fragmentation de mots-clés
  - antislash
  - slash
  - contournement de filtre
  - normalisation de chemin
---

# Insertion d'antislash et de slash

Deux particularités de normalisation permettent à un attaquant de réécrire un mot-clé ou un chemin filtré pour qu'il ne corresponde plus à une signature tout en résolvant toujours vers la même commande. Un **antislash** échappe le caractère suivant, et pour une lettre ordinaire le shell se contente de retirer l'antislash et de conserver la lettre. Les **slashs** redondants dans un chemin sont réduits par le résolveur de chemin du noyau. Les deux transformations se produisent après que le filtre a inspecté les octets bruts.

## Échappement par antislash d'un caractère ordinaire

En dehors des guillemets, `\x` est traité comme un `x` entre guillemets. Puisque mettre une simple lettre entre guillemets ne change rien à la lettre, `w\ho\am\i` se tokenise en `whoami` :

```
w\ho\am\i
c\at /etc/passwd
i\d
```

Les caractères antislash sont supprimés durant le retrait des guillemets, réassemblant le mot-clé. Une blocklist correspondant au littéral `whoami` ne voit que la forme échappée.

## Insertion de slash dans les chemins

Le résolveur de chemin traite les séquences de `/` comme un unique séparateur et ignore `/./`, de sorte qu'un chemin absolu peut être rembourré sans changer le fichier cible :

```
/bin/c\at /etc/passwd
//bin//cat /etc//passwd
/bin/./cat /etc/./passwd
```

`/bin/c\at` combine les deux astuces : l'antislash fragmente le nom du binaire tandis que le chemin explicite pointe toujours vers `cat`. Cela déjoue les filtres indexés sur `/bin/cat` ou sur le mot nu `cat`.

## Dans une requête HTTP

Encodez l'antislash en URL comme `%5c` là où l'application ne l'accepte pas brut :

```
w%5cho%5cam%5ci
```

## Combinaison avec d'autres évasions

Ces astuces ne dissimulent que le mot-clé ou le chemin. Ajoutez un substitut d'espace ou un séparateur là où c'est nécessaire :

```
/bin/c\at${IFS}/etc/passwd
127.0.0.1;w\ho\am\i
```

## Contexte et mises en garde

- Il s'agit d'une technique **shell** (`sh -c`, `system()`, backticks). Dans un appel `argv` pur, les antislashs sont des octets littéraux de l'argument et ne sont **pas** retirés.
- À l'intérieur de guillemets simples, un antislash est littéral, donc cette forme de fragmentation exige un contexte non entre guillemets ou entre guillemets doubles.
- L'échappement par antislash d'une *lettre* est distinct de la continuation antislash-**saut de ligne**, qui fusionne deux lignes ; voir la page dédiée aux continuations.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour élaborer des payloads à antislash et slash.
- **[commix](https://github.com/commixproject/commix)** : les scripts tamper automatisent les contournements par fragmentation de mots-clés.

## Références

- [PayloadsAllTheThings: Command Injection, bypass techniques](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [GTFOBins](https://gtfobins.github.io/)
