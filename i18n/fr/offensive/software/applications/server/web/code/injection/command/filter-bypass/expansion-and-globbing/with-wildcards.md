---
order: 2
title: "Contournement de filtre d'injection de commande par caractères génériques et globbing"
description: "Reconstruire des noms de binaires et des chemins filtrés avec les caractères glob du shell, /???/c?t /???/p?sswd, /bin/c*, pour que le littéral bloqué n'apparaisse jamais dans la requête ; inclut le comportement des caractères génériques sous Windows."
keywords:
  - injection de commande
  - caractère générique
  - globbing
  - contournement de filtre
  - reconstruction de chemin
  - évasion de WAF
---

# Caractères génériques

Le globbing du shell développe les caractères de motif, `?` (un caractère quelconque), `*` (une suite quelconque de caractères) et `[...]` (une classe de caractères), en les chemins de système de fichiers correspondants **avant** que la commande ne s'exécute. Cela permet à un attaquant de nommer un binaire ou un fichier cible sans taper son nom littéral : `/???/c?t /???/p?sswd` se développe en `/bin/cat /etc/passwd`, pourtant la requête ne contient ni `cat`, ni `passwd`, ni `/bin/`. Une liste de blocage qui fait correspondre ces littéraux ne se déclenche jamais.

## Pourquoi le shell le normalise

Le shell effectue l'**expansion de nom de chemin** sur tout mot non quoté contenant `?`, `*` ou `[` : il recherche dans les répertoires correspondants et remplace le motif par les chemins réels qui existent. `/???/c?t` correspond à `/bin/cat` (répertoire de trois caractères, nom de fichier `c`-quelconque-`t`). Parce que la substitution se produit à l'intérieur du shell après que le filtre a inspecté l'entrée, la requête ne transporte que des caractères génériques et des fragments partiels. La liste de blocage voit `/???/c?t` ; le noyau exécute `/bin/cat`. La sous-chaîne interdite est assemblée à partir du système de fichiers, pas à partir du payload.

## Payloads (POSIX)

Lire `/etc/passwd` lorsque `cat`, `passwd` ou les espaces plus le nom sont filtrés :

```
/???/c?t /???/p?sswd
/bin/c?t /etc/p?sswd
/???/??t /???/p??swd
/bin/c*t /e??/*sswd
```

Lancer des sondes d'identité sans le nom littéral du binaire :

```
/???/wh??mi
/bin/wh*
/usr/bin/i?
```

`/???/` correspond à tout répertoire de premier niveau à trois lettres, ce qui sous Linux se résout en `/bin` (et souvent `/sbin`, `/lib`, restreignez le motif suivant pour lever l'ambiguïté). Combinez avec une technique sans espace puisque les globs ont toujours besoin de séparateurs entre les arguments :

```
/???/c?t${IFS}/???/p?sswd
{/???/c?t,/???/p?sswd}
```

Une primitive particulièrement compacte abuse des répertoires `/bin/*` qui contiennent des outils aux noms prévisibles, par exemple en invoquant `tar`, `nc` ou un interpréteur par motif lorsque son nom littéral est bloqué :

```
/???/b??e64 /???/p?sswd        # base64 /etc/passwd
/usr/bin/p?rl -e '...'          # perl by glob
```

Le globbing aide aussi lorsque seul l'*argument* est filtré : développer un listing de répertoire fournit des noms de fichiers à un flag sans les nommer (`tar -cf /dev/stdout *` pour lire des fichiers via des primitives de type GTFOBins).

## Comportement des caractères génériques sous Windows

`cmd.exe` et PowerShell ne font **pas** de glob comme le font les shells POSIX. Le processeur de commandes ne développe pas `?`/`*` en listes d'arguments ; chaque programme effectue sa propre correspondance de caractères génériques sur les chemins qu'il reçoit. Conséquences pour les payloads :

- `type C:\???\*` ne se développe pas dans `cmd.exe`, `type` résout lui-même les caractères génériques, et seulement pour les arguments de fichier qu'il prend en charge.
- Les alias de nom court (8.3) sont l'analogue le plus proche : `C:\PROGRA~1` référence `Program Files` sans l'espace ni le nom complet.
- `Get-ChildItem` et les cmdlets PowerShell acceptent `*`/`?` comme leurs propres paramètres, mais le shell ne reconstruira pas un *nom* d'exécutable à partir d'un glob comme le fait `/???/c?t` sous Linux.

Traitez la reconstruction par caractère générique d'un nom de binaire comme une technique POSIX ; sous Windows, appuyez-vous plutôt sur les noms 8.3, les astuces de casse ou l'assemblage par variables d'environnement.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater pour concevoir des payloads basés sur le glob.
- **[GTFOBins](https://gtfobins.github.io/)** : binaires invocables par glob pour la lecture de fichiers et l'exécution.

## Références

- [PayloadsAllTheThings: Command Injection, Bypass without specific characters](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [GTFOBins](https://gtfobins.github.io/)
- [Bash Reference Manual: Filename Expansion](https://www.gnu.org/software/bash/manual/html_node/Filename-Expansion.html)
