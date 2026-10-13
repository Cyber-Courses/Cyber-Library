---
title: "Injection MVEL : exécution de code via le langage d'expression sur la JVM"
description: "MVEL évalue des expressions proches de Java avec un accès direct aux types Java, donc une expression influencée par l'attaquant passée à MVEL.eval ou à une expression compilée atteint Runtime et ProcessBuilder pour l'exécution de code."
keywords:
  - injection MVEL
  - MVEL.eval
  - langage d'expression MVEL
  - injection d'expression Java
  - RCE ProcessBuilder
---

# MVEL

MVEL (MVFLEX Expression Language) est un langage d'expression à l'exécution pour la JVM utilisé comme couche de script dans plusieurs frameworks et systèmes de règles : il sous-tend les actions de règle Drools, apparaît dans les prédicats de routage Spring et Camel, et est embarqué directement partout où une application veut un moteur de formules léger. Contrairement aux dialectes EL en sandbox, MVEL est proche de Java par conception et résout les références de type directement, donc il n'y a aucune restriction à contourner avant d'atteindre le runtime.

## Point d'entrée d'évaluation

Le puits est un appel qui compile et évalue une chaîne d'expression. La forme en un coup est `MVEL.eval(String)`, et la forme réutilisable compile une fois avec `MVEL.compileExpression(String)` et exécute le résultat au travers de `MVEL.executeExpression(compiled, context)`. Lorsque la chaîne remise à l'un ou l'autre appel porte des données de requête, la grammaire d'expression est contrôlée par l'attaquant :

```java
// expr taken from the request
Object result = MVEL.eval(expr);
```

## Exécution de code

MVEL lit les types Java pleinement qualifiés et appelle leurs méthodes statiques, donc `java.lang.Runtime` est atteignable sans préambule. Utilisez le nom pleinement qualifié : MVEL de base n'auto-importe pas `java.lang` (contrairement au code source Java, et contrairement à Drools qui ajoute des imports par défaut), donc un `Runtime` nu ne se résout pas sur un contexte de parseur par défaut. `Runtime.exec(String)` découpe la commande en tokens sur les espaces et l'exécute sans shell, donc il convient à un unique programme avec des arguments simples :

```java
java.lang.Runtime.getRuntime().exec("id")
```

Comme `exec(String)` n'a pas de shell, les pipes, `$(...)`, les redirections et les séparateurs `;` ne fonctionnent pas. Pour tout ce qui nécessite des fonctionnalités de shell, construisez un `ProcessBuilder` avec un tableau d'arguments explicite où le shell est le programme et la commande chaînée est un unique élément :

```java
new java.lang.ProcessBuilder(new String[]{"/bin/bash","-c","id; uname -a"}).start()
```

MVEL peut aussi créer des objets et invoquer des méthodes d'instance, donc la sortie est lisible dans la même expression en vidant le flux du processus :

```java
new java.util.Scanner(new java.lang.ProcessBuilder(new String[]{"/bin/bash","-c","id"}).start().getInputStream()).useDelimiter("\\A").next()
```

MVEL accepte également la syntaxe d'import avec `@` en tête, et les noms de classe non qualifiés se résolvent lorsque le type est déjà importé dans le contexte du parseur, ce qui raccourcit les payloads quand l'application précharge des packages courants. Là où seule une navigation de propriété est évaluée plutôt qu'une instruction complète, la même voie de réflexion utilisée par les dialectes unified EL s'applique : atteindre `getClass().forName(...)` depuis n'importe quel objet en portée et construire le `ProcessBuilder` au travers de ses constructeurs déclarés.

## Outils

- **Burp Suite** : Repeater pour délivrer des payloads MVEL.eval et lire la sortie.
- **J2EEScan** : extension Burp avec des contrôles d'injection d'expression Java.
- Payloads MVEL manuels nommant java.lang.Runtime et ProcessBuilder.

## Références

- [MVEL Language Guide](http://mvel.documentnode.com/)
- [PayloadsAllTheThings: Java EL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
