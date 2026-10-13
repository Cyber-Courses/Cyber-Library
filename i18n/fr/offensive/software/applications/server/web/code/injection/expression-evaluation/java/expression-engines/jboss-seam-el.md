---
title: "Injection JBoss Seam EL : exécution de commandes à distance via l'EL unifié"
description: "Seam évalue des expressions EL unifiées, et une EL contrôlée par l'attaquant atteignant une valeur interpolée ou un paramètre de vue résout du Java arbitraire, donnant l'exécution de commandes dans le serveur d'applications."
keywords:
  - JBoss Seam
  - injection Seam EL
  - EL unifié
  - injection de langage d'expression
  - exécution de commandes à distance
---

# JBoss Seam EL

JBoss Seam s'appuie sur l'Unified Expression Language (`#{...}`), en l'utilisant bien plus agressivement qu'une simple couche de vue : l'EL pilote la navigation, les actions de page, les paramètres et les messages interpolés à travers le framework. Seam a aussi étendu l'EL avec des appels de méthodes paramétrés. La conséquence de sécurité est que tout endroit où Seam évalue une chaîne EL construite à partir d'une entrée non fiable devient une voie vers du Java arbitraire, car la résolution EL atteint les méthodes d'objets et, à travers elles, le runtime. C'est le mécanisme derrière l'exposition bien connue d'exécution de commandes à distance de Seam, où une expression EL délivrée via un paramètre de requête est évaluée côté serveur.

## Pattern vulnérable

Seam évalue l'EL dans plusieurs sinks : les valeurs d'`actionOutcome` et d'action de page, les messages de statut et de faces interpolés, et le code applicatif qui résout directement une expression :

```java
// value derived from the request
Expression expr = expressions.createValueExpression(input);
Object result = expr.getValue();
```

Lorsque `input` porte de l'EL de l'attaquant, le résolveur de Seam la parcourt contre tout le contexte applicatif, y compris les objets implicites que Seam expose.

## Atteindre le runtime

L'EL unifié n'a pas d'`import`, pas de `new` et pas de syntaxe de littéral de classe, donc il ne peut pas nommer `java.lang.Runtime` directement. La voie atteignable utilise le support des appels de méthodes de Seam pour passer par la réflexion depuis un littéral de chaîne : obtenir `Class` via `forName`, chercher la méthode statique `getRuntime`, l'invoquer avec un récepteur null, puis appeler `exec` :

```
#{''.getClass().forName('java.lang.Runtime').getMethod('getRuntime').invoke(null).exec('id')}
```

Relire la sortie enveloppe le flux `Process` renvoyé à travers d'autres appels EL réflexifs, construisant un reader ou un scanner sur `getInputStream()` pour que le résultat de la commande atterrisse dans la valeur que Seam rend.

Historiquement, cette surface était atteinte par des points d'entrée non authentifiés tels que le paramètre de requête `actionOutcome` sur les pages par défaut de Seam, où la valeur du paramètre est prise comme un outcome EL et évaluée pendant la navigation, de sorte qu'aucun code applicatif n'a à y consentir pour que l'expression s'exécute.

## Fonctionnalités de shell

`Runtime.exec(String)` tokenise sur les espaces sans shell, donc `$(...)`, les pipes et les redirections ne s'étendent pas, et la forme à chaîne unique ci-dessus exécute un seul binaire. L'EL unifié ne peut pas écrire un littéral `String[]` (pas de `new`, pas de syntaxe de tableau), donc façonner une invocation `/bin/bash -c` signifie construire le tableau d'arguments par réflexion via `java.lang.reflect.Array` et le passer à la surcharge `exec(String[])`, ce qui est bien plus verbeux que la forme à commande unique. En pratique, redirigez la sortie de la commande unique vers un fichier et relisez-la, ou utilisez la commande one-shot là où aucune fonctionnalité de shell n'est requise.

## Outils

- **Burp Suite** : Repeater pour délivrer des payloads EL via actionOutcome et d'autres sinks.
- **J2EEScan** : extension Burp qui signale l'injection Java EL.
- Payloads manuels de réflexion en EL unifié atteignant Runtime.

## Références

- [Jakarta Expression Language specification](https://jakarta.ee/specifications/expression-language/)
- [JBoss Seam reference documentation](https://docs.jboss.org/seam/latest/reference/html_single/)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
