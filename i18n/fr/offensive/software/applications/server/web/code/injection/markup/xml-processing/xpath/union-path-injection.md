---
order: 2
title: "Injection de chemin par union XPath"
description: "L'opérateur d'union | ajoute des chemins de localisation arbitraires à un XPath construit par concaténation de chaînes, sélectionnant des nœuds hors du sous-arbre prévu et dumpant l'ensemble du document."
keywords:
  - injection par union XPath
  - opérateur pipe XPath
  - injection de chemin de localisation
  - union de node-set
  - dump de document XML
---

# Injection de chemin par union

L'opérateur `|` de XPath renvoie l'union de deux node-sets. Là où l'injection SQL recourt à `UNION SELECT`, l'injection XPath recourt à `|`, et c'est plus simple : il n'y a pas de nombre de colonnes à faire correspondre ni de schéma à satisfaire, donc un pipe injecté peut ajouter n'importe quel chemin de localisation et replier ses résultats dans la sortie de la requête.

## L'opérateur

Étant donné une requête qui renvoie un node-set contraint, fermer le prédicat et ajouter `| <path>` ajoute un second chemin choisi par l'attaquant. Une requête telle que `//user[name='x']/pass` peut être transformée en une union qui renvoie aussi des nœuds sans rapport :

```
x']|//*|//user['
```

L'expression devient `//user[name='x']|//*|//user['']/pass`. Le terme du milieu `//*` sélectionne chaque élément du document, donc le jeu de résultats contient désormais l'arbre entier plutôt qu'un seul nœud de mot de passe. Les fragments de début et de fin n'existent que pour maintenir la syntaxe environnante bien formée.

## Dumper l'ensemble du document

`//*` est le sélecteur le plus large et renvoie chaque nœud d'élément. Quand l'application reflète les résultats de requête dans la page, cela affiche le document :

```
']|//*|//foo[bar='
```

Pour un dump plus lisible, ciblez explicitement les nœuds de texte et d'attribut :

```
']|//*/text()|//foo[bar='
']|//@*|//foo[bar='
```

`//@*` énumère chaque attribut de tout le document, ce qui contient souvent des rôles, des identifiants et des drapeaux qui ne sont pas présents en tant que texte d'élément.

## Atteindre des sous-arbres spécifiques

L'union ne doit pas forcément être indiscriminée. Ajoutez un chemin précis pour extraire une branche d'intérêt tout en gardant la requête d'origine valide :

```
']|//user/pass|//foo[bar='
']|//config/credentials|//foo[bar='
```

Chaque chemin injecté est indépendant des contraintes de la requête d'origine, donc un filtre qui limitait le premier node-set à un seul utilisateur est sans rapport avec le chemin unifié.

## Axes parent et ancêtre

Quand le point d'injection se situe profondément dans l'arbre, les axes parent (`..`) et ancêtre remontent vers la racine pour atteindre des données de nœuds frères que la requête d'origine a dépassées. Cela récupère des nœuds au-dessus ou à côté de la sélection prévue :

```
']|../../*|//foo[bar='
']|ancestor-or-self::*|//foo[bar='
']|//user[1]/following-sibling::user|//foo[bar='
```

`following-sibling` et `preceding-sibling` parcourent les enregistrements au même niveau, ce qui est un moyen efficace d'énumérer chaque `user` une fois que l'un est accessible. Les axes ancêtre sont le complément vertical : depuis tout nœud correspondant, ils exposent ses conteneurs jusqu'à la racine du document.

## Garder l'expression valide

Le motif récurrent est `<fermeture> | <chemin injecté> | <réouverture>`, où le fragment de fermeture termine le littéral et le prédicat d'origine et le fragment de réouverture fournit un prédicat factice que le parseur peut terminer. Si l'application rejette la requête d'emblée plutôt que de renvoyer des nœuds supplémentaires, l'équilibre est faux ; comptez les guillemets et crochets ouverts dans la requête d'origine et reproduisez-les. Une union mal formée produit généralement une erreur de parseur, que l'extraction basée sur les erreurs peut ensuite exploiter à part entière.

## Outils

- **Burp Repeater** : ajout de chemins d'union `|` et lecture des nœuds supplémentaires.
- **xcat** : automatisation de l'énumération de node-set à travers le document.
- Tests manuels avec des charges utiles `|//*` et basées sur les axes.

## Références

- [OWASP: XPATH Injection](https://owasp.org/www-community/attacks/XPATH_Injection)
- [PayloadsAllTheThings: XPATH Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)
