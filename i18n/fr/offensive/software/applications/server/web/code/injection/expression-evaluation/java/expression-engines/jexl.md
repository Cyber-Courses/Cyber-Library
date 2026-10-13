---
title: "Injection JEXL : exécution de code via Apache Commons JEXL"
description: "Les expressions Apache Commons JEXL peuvent résoudre des classes, construire des objets et invoquer des méthodes, donc une entrée non fiable dans une évaluation JEXL atteint ProcessBuilder et le runtime de la JVM pour l'exécution de commandes."
keywords:
  - JEXL
  - injection JEXL
  - Apache Commons JEXL
  - opérateur new()
  - exécution de code JVM
---

# JEXL

Apache Commons JEXL est un langage de scripting et d'expression pour la JVM, utilisé pour laisser la configuration, des règles ou des formules fournies par l'utilisateur s'exécuter contre les objets applicatifs. Contrairement à un évaluateur purement mathématique, JEXL est conçu pour interagir avec Java : les expressions peuvent lire et écrire des variables de contexte, appeler des méthodes sur tout objet qu'elles peuvent atteindre, indexer des collections, et, via l'opérateur `new`, construire des classes arbitraires par leur nom. Cette conception est exactement ce qui fait d'une expression JEXL injectée un sink d'exécution de code plutôt qu'un problème de projection de données.

## Pattern vulnérable

```java
JexlEngine jexl = new JexlBuilder().create();
// expression text from the request
JexlExpression e = jexl.createExpression(input);
Object result = e.evaluate(context);
```

Tout ce que l'hôte place dans le `JexlContext` est atteignable, et parce que JEXL résout les noms de classes, l'attaquant n'est pas limité aux objets enregistrés.

## Atteindre le runtime

L'opérateur `new` de JEXL instancie une classe à partir de son nom pleinement qualifié, et les appels de méthodes s'enchaînent sur le résultat. Un `ProcessBuilder` construit et démarré ainsi exécute une commande dans le processus de l'application :

```
new('java.lang.ProcessBuilder', ['id']).start()
```

Une alternative pivote depuis la classe de n'importe quel objet pour atteindre `Runtime` par réflexion. La méthode statique `getRuntime` n'est pas une méthode de l'objet `Class` que `forName` renvoie, donc on l'atteint avec `getMethod(...).invoke(null)` plutôt qu'en la chaînant directement :

```
''.getClass().forName('java.lang.Runtime').getMethod('getRuntime').invoke(null).exec('id')
```

Pour relire la sortie de la commande dans la réponse, enveloppez le flux du processus démarré, par exemple en construisant un `java.util.Scanner` sur le flux d'entrée du processus et en lisant un token délimité, le tout exprimable à travers la même grammaire `new`/appel de méthode.

## Les fonctionnalités de shell nécessitent un vecteur d'arguments

`Runtime.exec(String)` et un argument `ProcessBuilder` à chaîne unique tokenisent sur les espaces et s'exécutent sans shell, donc `$(...)`, les backticks, les pipes et les redirections sont passés comme arguments littéraux et ne s'étendent jamais. Pour tout ce qui nécessite un shell, construisez explicitement la liste d'arguments et laissez `/bin/bash -c` l'interpréter :

```
new('java.lang.ProcessBuilder', ['/bin/bash','-c','id > /tmp/o 2>&1']).start()
```

Sous Windows le vecteur équivalent est `['cmd.exe','/c','whoami > C:\\Windows\\Temp\\o.txt']`.

## Comportement selon la version

La grammaire atteignable dépend de la version majeure de JEXL et de la façon dont l'hôte a configuré le moteur. JEXL 2 évalue par défaut cette grammaire résolvant les classes et appelant les méthodes. JEXL 3 a introduit un modèle de permissions et de sandbox plus explicite (`JexlPermissions`, `JexlContext` restreint et résolveurs de namespace), donc le fait que `new` et la résolution de classe arbitraire soient atteignables dépend de si l'application a laissé les valeurs par défaut permissives ou a verrouillé le moteur. Là où un moteur JEXL 3 est restreint, la surface atteignable se réduit aux méthodes et objets que l'hôte a explicitement autorisés, donc énumérez ce qui se résout (construction de classe, `getClass`, fonctions de namespace enregistrées) avant de vous engager dans un payload de commande complet. La variante template (`JxltEngine`, `${...}` / `#{...}`) enveloppe la même grammaire d'expression, donc un sink `${...}` porte la même portée.

## Outils

- **Burp Suite** : Repeater pour délivrer des payloads JEXL new()/réflexion.
- **J2EEScan** : extension Burp avec des contrôles d'injection d'expression Java.
- Payloads JEXL manuels construisant ProcessBuilder ou réfléchissant vers Runtime.

## Références

- [Apache Commons JEXL](https://commons.apache.org/proper/commons-jexl/)
- [Apache Commons JEXL: syntax reference](https://commons.apache.org/proper/commons-jexl/reference/syntax.html)
- [Apache Commons JEXL: permissions and sandboxing](https://commons.apache.org/proper/commons-jexl/apidocs/org/apache/commons/jexl3/introspection/JexlPermissions.html)
