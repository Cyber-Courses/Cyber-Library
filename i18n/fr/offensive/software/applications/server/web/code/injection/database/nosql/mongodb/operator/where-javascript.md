---
order: 5
title: "Injection de JavaScript côté serveur MongoDB via $where et mapReduce"
description: "$where et mapReduce évaluent du JavaScript côté serveur ; là où il est activé, des chaînes JS injectées s'échappent des expressions, forcent des conditions toujours vraies et expirent en aveugle."
keywords:
  - $where
  - mapReduce
  - JavaScript côté serveur
  - injection MongoDB
  - évasion de chaîne JavaScript
  - time-based aveugle
---

# Where JavaScript

`$where` et `mapReduce` évaluent du JavaScript **côté serveur** sur chaque document, ce qui fait de toute entrée utilisateur atteignant leur chaîne de code un sink d'injection JavaScript plutôt qu'un simple sink d'opérateur.

## Précondition de disponibilité

Le JavaScript côté serveur est **activé par défaut** : `security.javascriptEnabled` vaut `true` par défaut, de sorte que `$where`, `mapReduce` et `$function` s'exécutent sauf si un opérateur a explicitement désactivé le scripting. Il est déprécié dans les versions récentes et certains déploiements durcis le désactivent, donc traitez l'exécution comme la valeur par défaut mais confirmez-la sur la cible. `$where` ne peut pas utiliser d'opérateurs de requête et doit être une expression ou une fonction JavaScript, de sorte que ce sink est atteignable là où l'application construit une chaîne `$where` (ou passe l'entrée dans `mapReduce`/`$function`) à partir de l'entrée. Lorsqu'un opérateur a désactivé le scripting, ces payloads échouent et les techniques d'opérateur ailleurs dans ce sous-arbre sont la voie.

## Le sink

```javascript
// user input concatenated into a $where string
db.collection("users").find({
  $where: "this.username == '" + req.body.username + "'"
});
```

L'entrée atterrit à l'intérieur d'une chaîne JavaScript comparée par document, donc un guillemet ferme la chaîne et le reste est analysé comme du JavaScript.

## Évasion de chaîne JavaScript

Fermez le littéral et forcez toute l'expression à vrai, exactement comme dans l'évasion de chaîne SQL mais en syntaxe JavaScript :

```
' || '1'=='1
' || true || '
a'; return true; var x='
```

Le premier rend le prédicat par document `this.username == '' || '1'=='1'`, vrai pour chaque document, renvoyant toute la collection. Un `$where` à valeur de fonction est remplacé en bloc lorsque l'entrée le contrôle :

```javascript
{ "$where": "function(){ return true; }" }
```

Comme `$where` exécute du vrai JavaScript, l'expression peut lire d'autres champs du document courant pour exfiltrer via un oracle booléen :

```
' || this.password[0]=='a' || '
' || this.role=='admin' || '
```

Chaque requête teste une condition sur un autre champ, et le signal match/no-match récupère la valeur caractère par caractère, le même oracle en boucle utilisé avec `$regex` mais exprimé en JavaScript.

## Time-based aveugle via une boucle d'attente active

Lorsqu'aucune différence de résultat n'est observable mais que le JavaScript est activé, retardez l'exécution et mesurez le temps de réponse. Le scope `$where` côté serveur n'expose pas le helper de shell `sleep()` (ce global n'existe que dans le shell mongo), de sorte qu'un appel `sleep(5000)` lève `ReferenceError` au lieu de mettre en pause. La primitive portable est une attente active CPU qui tourne jusqu'à une échéance d'horloge murale, exécutée seulement lorsque la condition sondée est vraie :

```
' || (this.username=='admin' && (function(){var t=Date.now();while(Date.now()-t<5000){}return true})()) || '
```

```javascript
{ "$where": "function(){ if (this.role=='admin') { var t=Date.now(); while(Date.now()-t<5000){} } return false; }" }
```

Une réponse lente signifie que la condition était vraie. Bornez la boucle en ancrant sur un champ à la fois (`this.secret[0]=='a'`, puis `[1]`, et ainsi de suite), en inférant chaque caractère selon que la requête se bloque ou non. Comme `$where` s'exécute contre chaque document du scan, maintenez l'ensemble matché petit (épinglez d'abord un nom d'utilisateur) pour que le délai soit attribuable et que le scan reste bon marché.

## mapReduce

`mapReduce` exécute des fonctions JavaScript `map` et `reduce` côté serveur et est soumis à la même porte `javascriptEnabled`. Lorsque l'entrée atteint le corps de ces fonctions, la même évasion et la même inférence s'appliquent, avec les clés/valeurs émises comme canal de sortie :

```javascript
db.collection.mapReduce(
  "function(){ emit(this.username, this.password); }",  // injected map body
  "function(k,v){ return v; }",
  { out: { inline: 1 } }
);
```

Un `map` injecté qui émet des champs sensibles transforme la sortie du reduce en canal d'extraction de masse lorsque le résultat est reflété.

## Outils

- **mongosh** : tester les payloads JavaScript $where et mapReduce contre une base de données.
- **Burp Repeater** : livrer des payloads d'évasion JavaScript et time-based via le sink.
- **NoSQLMap** : automatiser la détection et l'extraction d'injection $where.

## Références

- [MongoDB : $where](https://www.mongodb.com/docs/manual/reference/operator/query/where/)
- [MongoDB : Server-side JavaScript](https://www.mongodb.com/docs/manual/core/server-side-javascript/)
- [MongoDB : mapReduce](https://www.mongodb.com/docs/manual/reference/command/mapReduce/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
