---
order: 3
title: "XQuery basé sur les erreurs : inférer des données à partir des erreurs remontées"
description: "Les erreurs statiques et dynamiques XQuery renvoyées au client fuitent du contenu, des types et de la structure via des conditions d'erreur contrôlées et des données intégrées dans les chaînes d'erreur."
keywords:
  - XQuery basé sur les erreurs
  - messages d'erreur XQuery
  - inférence d'erreur dynamique
  - fuite d'erreur de type
  - extraction XQuery aveugle
---

# Basé sur les erreurs

Lorsqu'une application renvoie les erreurs du processeur au client, l'injection XQuery n'a pas besoin de résultats reflétés pour extraire des données. Le texte de l'erreur, le code d'erreur et la simple présence ou absence d'une erreur portent chacun de l'information, et un attaquant façonne l'expression injectée de sorte que la condition testée décide lequel de ces résultats survient. Cela transforme une seule requête en un oracle fiable même sur des points d'accès qui ne rendent qu'une page d'échec générique, à condition que le statut ou le corps diffère entre succès et erreur.

## Données intégrées dans la chaîne d'erreur

La technique la plus directe force le moteur à placer les données cibles dans un message qu'il affiche ensuite. Caster une chaîne dérobée vers un type qui ne peut pas la contenir fait citer la valeur fautive par le processeur dans une erreur dynamique :

```xquery
' or (//user[1]/password) cast as xs:integer or '
```

Parce qu'un mot de passe n'est pas un entier valide, le moteur lève `FORG0001` et de nombreux déploiements incluent la valeur rejetée dans le message, fuitant le secret verbatim. `fn:error()` avec une description construite est encore plus explicite là où l'application transmet un texte d'erreur personnalisé :

```xquery
' or error(xs:QName('err:LEAK'), string-join((//user/password), '|')) or '
```

## Erreur conditionnelle comme oracle booléen

Quand les messages sont nettoyés mais que les erreurs changent encore la réponse, on fait déclencher par un prédicat une faute d'exécution garantie seulement sur une condition vraie. La division par zéro et l'accès à une sous-séquence hors limites sont des fautes pratiques :

```xquery
' or (1 div (if (substring((//user[1]/password),1,1)='a') then 0 else 1)) or '
```

Une supposition vraie divise par zéro et lève `FOAR0001`, produisant la page d'erreur ; une supposition fausse s'évalue proprement et renvoie la page normale. Itérer `substring()` à travers les positions reconstruit la valeur un caractère par requête, la même recherche que dans l'injection booléenne aveugle mais indexée sur le signal d'erreur au lieu du jeu de résultats.

## Sondage de type et de structure

Les erreurs de type statiques révèlent la forme des données sans les extraire. Appliquer une fonction réservée aux nœuds à un élément lié, ou comparer des types incompatibles, indique à l'attaquant si un chemin produit un élément, une valeur atomique ou une séquence vide :

```xquery
' or name(//user[1]/*[1]) = 'x' or fn:local-name(0) or '
```

Le code spécifique renvoyé (`XPTY0004` pour une incompatibilité de type, `XPST0017` pour une fonction inconnue ou une arité incorrecte) identifie aussi le moteur et sa version de XQuery, puisque BaseX, eXist-db, Saxon et MarkLogic émettent une formulation et des préfixes distincts autour des mêmes codes W3C. Cette empreinte guide les fonctions d'extension propriétaires à tenter ensuite.

## Chaînes d'erreur propriétaires

Les messages spécifiques au moteur divulguent fréquemment des chemins du système de fichiers, des URI de module et des noms de variables internes. Un appel `doc()` ou `file:read-text()` délibérément mal formé provoque une erreur de ressource introuvable dont le texte renvoie le chemin absolu résolu, cartographiant la disposition du déploiement pour une charge utile de lecture de fichier ultérieure.

## Outils

- **Burp Repeater** : forcer des fautes de cast et de division par zéro et lire le texte d'erreur.
- Tests manuels avec des charges utiles `cast as xs:integer` et `fn:error()`.

## Références

- [W3C XQuery 3.1: Error Handling](https://www.w3.org/TR/xquery-31/#id-error-handling)
- [PayloadsAllTheThings: XPath Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)
