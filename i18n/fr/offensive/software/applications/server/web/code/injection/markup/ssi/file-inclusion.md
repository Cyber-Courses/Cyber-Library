---
title: "Inclusion de fichier SSI"
description: "La directive include intègre des fichiers et des ressources côté serveur dans une page interprétée en SSI ; les chemins file et virtual ainsi que la traversée permettent à un attaquant de lire des fichiers locaux et d'atteindre des points d'accès internes."
keywords:
  - SSI include file
  - include virtual
  - traversée de chemin SSI
  - inclusion de ressource côté serveur
  - lecture de fichier local
---

# Inclusion de fichier

La directive `include` insère le contenu d'une autre ressource dans la page au moment du parsing. Lorsque l'entrée attaquant atteint un document interprété en SSI, un `include` injecté lit des fichiers sur le disque ou intègre des ressources côté serveur, et la traversée de chemin dans le paramètre élargit cette portée à des fichiers situés hors de la racine du document.

## Les deux formes

`include` prend l'un de deux paramètres, et la différence décide de ce que le serveur résout.

```
<!--#include file="footer.html"-->
<!--#include virtual="/includes/footer.html"-->
```

`file` nomme un chemin relatif au document courant et, par défaut, n'autorise pas `..` pour remonter au-dessus de celui-ci, bien que les mauvaises configurations et les particularités du parseur assouplissent fréquemment cette règle. `virtual` prend un chemin dans l'espace d'URL, relatif à la racine du document, et est résolu par le serveur, donc il peut atteindre tout ce que le serveur web mappe, y compris les gestionnaires dynamiques et la sortie CGI.

## Lire des fichiers locaux

`file` est la voie vers le système de fichiers, mais sur le `mod_include` standard il se résout relativement au document courant et rejette un chemin absolu, donc atteindre un fichier système signifie en sortir par traversée plutôt que de le nommer directement :

```
<!--#include file="../../../../../../etc/passwd"-->
```

Sur des cibles Windows, ajustez les séparateurs et les cibles :

```
<!--#include file="..\..\..\..\..\..\windows\win.ini"-->
```

Une traversée encodée peut passer outre des filtres d'entrée qui ne correspondent qu'au `../` littéral :

```
<!--#include file="..%2f..%2f..%2f..%2f..%2f..%2fetc/passwd"-->
```

`virtual` se résout dans l'espace d'URL, pas sur le disque, donc il ne lit un chemin OS comme `/etc/passwd` que là où un alias ou un mappage explicite l'expose ; sa véritable portée concerne les ressources que le serveur mappe, abordées ci-après.

## Inclure des ressources côté serveur

Parce que `virtual` est résolu dans l'espace d'URL, il atteint des points d'accès dynamiques et pas seulement des fichiers statiques. Inclure un script côté serveur intègre la sortie rendue de ce point d'accès dans la réponse, ce qui peut exposer des pages réservées en interne ou déclencher leurs effets de bord dans le contexte du serveur :

```
<!--#include virtual="/admin/status"-->
<!--#include virtual="/cgi-bin/internal.cgi?debug=1"-->
<!--#include virtual="/server-status"-->
```

Un `include virtual` est une sous-requête interne, pas une nouvelle connexion HTTP depuis localhost, et il conserve la requête et le contexte client d'origine pour les contrôles d'accès, donc il ne falsifie pas une origine loopback ni ne contourne une restriction d'adresse source. Ce qu'il gagne, c'est d'atteindre des ressources dans l'espace d'URL qui ne sont pas liées dans la navigation normale, d'intégrer leur sortie rendue dans la réponse et de déclencher leurs effets de bord pendant l'assemblage de la page. Là où le gestionnaire inclus reflète ses propres paramètres, l'inclusion peut aussi devenir un pivot vers une injection supplémentaire.

## Confirmer et itérer

Commencez par une inclusion connue comme présente pour confirmer que la directive est interprétée, puis passez aux cibles intéressantes :

```
<!--#include virtual="/robots.txt"-->
```

Une réponse qui contient désormais le corps de `robots.txt` confirme la résolution. Utilisez le chemin sur disque divulgué par `echo var="SCRIPT_FILENAME"` ou `PATH_TRANSLATED` pour calculer le nombre exact d'étapes de traversée nécessaires, ce qui supprime le tâtonnement dans l'empilement des séquences `../`.

Si `include` renvoie une erreur plutôt que du contenu (par exemple `[an error occurred while processing this directive]`), la directive a été interprétée mais le chemin a échoué ; ajustez le chemin ou basculez entre `file` et `virtual`.

## Outils

- **Burp Suite** : injection de directives `include` et itération de la profondeur de traversée avec Intruder.
- Tests manuels avec des charges utiles `<!--#include file-->` et `<!--#include virtual-->` forgées.

## Références

- [OWASP: Server-Side Includes (SSI) Injection](https://owasp.org/www-community/attacks/Server-Side_Includes_(SSI)_Injection)
- [PayloadsAllTheThings: Server Side Inclusion / Edge Side Inclusion Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Include%20Injection)
