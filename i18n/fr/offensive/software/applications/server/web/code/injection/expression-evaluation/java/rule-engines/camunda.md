---
title: "Injection Camunda : exécution de code via les expressions JUEL et les tâches de script"
description: "Camunda BPMN évalue des expressions EL unifiées liées à des méthodes Java et exécute du code de tâches de script, donc une entrée non fiable dans une expression de processus invoque des méthodes et atteint le runtime de la JVM."
keywords:
  - Camunda
  - injection Camunda
  - JUEL
  - expression BPMN
  - tâche de script
---

# Camunda

Camunda est un moteur de processus BPMN et de décision. Les modèles de processus évaluent des expressions à de nombreux points : conditions de flux de séquence, expressions de délégation de tâche de service, mappings d'entrée et de sortie, listeners, et tables de décision DMN. Le langage d'expression par défaut est l'EL unifié (JUEL), `${...}`, et Camunda lie la résolution EL aux variables de processus et aux beans Spring ou CDI, ce qui signifie qu'une expression peut invoquer des méthodes Java. Lorsqu'une définition de processus, une valeur de formulaire ou une variable qui alimente une expression est influencée par l'attaquant, l'EL injectée s'exécute dans le moteur et atteint le runtime via la résolution de méthode.

## Pattern vulnérable

L'exposition nécessite le contrôle par l'attaquant du **texte** de l'expression, et pas simplement d'une variable que l'expression lit. Une variable de processus interpolée avec `${someVar}` renvoie la valeur de la variable comme donnée ; cette valeur n'est pas re-parsée récursivement comme de l'EL, donc contrôler la variable seule n'est pas une injection. Les vrais points d'entrée sont là où un texte non fiable devient l'expression elle-même :

```xml
<!-- untrusted input concatenated INTO the condition expression -->
<conditionExpression xsi:type="tFormalExpression">${amount > USERINPUT}</conditionExpression>
```

ou un modèle déployable que l'attaquant peut rédiger, ou une chaîne d'expression remise à l'API REST de Camunda (le moteur évalue directement les champs de condition et d'expression qu'on lui donne), ce qui est la voie la plus propre parce que l'attaquant fournit le texte EL verbatim.

## Atteindre le runtime via JUEL

JUEL (EL unifié standard) n'a pas d'`import`, pas de `new`, pas de littéraux de tableau et pas de syntaxe de littéral de classe, donc il ne peut pas nommer `java.lang.Runtime` directement à moins que l'application n'ait exposé un bean sous ce nom. La voie portable atteint la classe par réflexion depuis un littéral de chaîne, puis invoque le `getRuntime` statique :

```
${''.getClass().forName('java.lang.Runtime').getMethod('getRuntime').invoke(null).exec('id')}
```

La sortie est relue en enveloppant le flux de processus renvoyé à travers d'autres appels réflexifs dans la même grammaire EL.

## Les tâches de script sont une voie plus directe

Si l'attaquant contrôle une tâche de script ou une expression de script inline, Camunda exécute le langage nommé directement via JSR-223, Groovy et JavaScript étant couramment sur le classpath. C'est une exécution immédiate sans le chaînage de méthodes EL :

```groovy
"id".execute().text
```

Une tâche de script JavaScript (Nashorn/Graal) atteint `java.lang.Runtime` de la même façon. Parce que les tâches de script exécutent le code de l'attaquant sans indirection EL, elles sont le sink le plus propre partout où le modèle de processus ou une ressource de script est contrôlable.

## Les fonctionnalités de shell nécessitent une tâche de script

`Runtime.exec(String)` tokenise sur les espaces sans shell, donc `$(...)`, les pipes et les redirections restent littéraux, et la voie EL réflexive ci-dessus exécute un seul binaire. JUEL ne peut pas construire un vecteur d'arguments `String[]` (il n'a ni `new` ni littéral de tableau), donc façonner une invocation de shell via de l'EL pure est impraticable. Là où des fonctionnalités de shell sont nécessaires, la voie de la tâche de script est la route : un script Groovy construit une liste explicite et l'exécute via un shell.

```groovy
["/bin/bash","-c","id | base64 > /tmp/o"].execute()
```

Sous Windows la liste est `["cmd.exe","/c","whoami"]`.

## Outils

- **Burp Suite** : Repeater pour soumettre des payloads EL et de tâche de script via l'API REST.
- Payloads JUEL manuels de réflexion ; les tâches de script exécutent Groovy ou JavaScript directement.

## Références

- [Camunda: expression language](https://docs.camunda.org/manual/latest/user-guide/process-engine/expression-language/)
- [Camunda: scripting](https://docs.camunda.org/manual/latest/user-guide/process-engine/scripting/)
- [Jakarta Expression Language specification](https://jakarta.ee/specifications/expression-language/)
