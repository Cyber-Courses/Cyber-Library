---
title: "Injection de langage d'expression Java : moteurs, systèmes de règles et frameworks web"
description: "Comment un texte contrôlé par l'attaquant atteignant un langage d'expression JVM transforme une fonctionnalité applicative en invocation de méthode et, sur les moteurs avec accès au runtime, en exécution de code."
keywords:
  - injection d'expression Java
  - langage d'expression
  - injection EL
  - injection de moteur de règles
  - exécution de code JVM
---

# Java

La JVM héberge une large famille de langages d'expression. Certains sont embarqués directement dans des frameworks (EL unifié dans Jakarta/BPMN, SpEL dans Spring), certains sont des bibliothèques d'évaluation autonomes qu'une application intègre pour laisser les utilisateurs écrire des formules ou des règles (JEXL, MVEL, Exp4j, mXparser), et certains pilotent des produits de routage ou de règles (Camel Simple, Drools, Camunda). Ils partagent une exposition commune : lorsque le code applicatif construit une chaîne d'expression à partir d'une entrée non fiable et la remet à un évaluateur, l'attaquant écrit le programme que l'évaluateur exécute.

L'impact atteignable n'est pas uniforme, et se tromper gâche une mission. Les langages se divisent en deux groupes :

- **Les langages atteignant le runtime** résolvent les types, construisent des objets et appellent des méthodes. Une expression injectée atteint `java.lang.Runtime`, `ProcessBuilder` et la réflexion, donc le résultat est l'exécution de code dans le processus de l'application. EL (JUEL/Seam), SpEL, OGNL, MVEL (et Drools bâti dessus), JEXL, Camel Simple et Camunda JUEL appartiennent tous ici.
- **Les évaluateurs mathématiques sandboxés** parsent des nombres, des opérateurs et seulement les fonctions et variables que l'hôte a enregistrées. Exp4j et mXparser n'ont aucune voie vers les types Java, la réflexion ou le runtime, donc ils ne donnent pas d'exécution de code inhérente. Leur surface injectable est la grammaire mathématique, toutes fonctions ou variables personnalisées que l'application a exposées, et l'abus par épuisement de ressources. Traiter ceux-ci comme des sinks de RCE est une erreur.

À travers chaque moteur JVM atteignant le runtime, le sink de commande est le même, et son unique écueil aussi. `Runtime.exec(String)` découpe son argument sur les espaces et exécute le résultat directement sans shell, donc `$(...)`, les backticks, les pipes, `>` et `&&` sont passés comme arguments littéraux et ne s'étendent jamais. Tout payload qui a besoin de fonctionnalités de shell construit à la place un vecteur d'arguments explicite :

```java
new String[]{"/bin/bash","-c","id > /tmp/o 2>&1"}
```

et l'exécute via `ProcessBuilder` ou la surcharge `Runtime.exec(String[])`. Les pages ci-dessous appliquent cela dans la syntaxe native de chaque moteur.

Cette section est organisée en trois groupes enfants :

- [Moteurs d'expression](expression-engines/index.md) : bibliothèques d'évaluation autonomes qu'une application embarque pour des formules, des templates ou du scripting : Exp4j, JBoss Seam EL, JEXL et mXparser.
- [Moteurs de règles](rule-engines/index.md) : produits de règles, de routage et de workflow dont les expressions et conséquences s'exécutent sur une entrée non fiable : Apache Camel, Camunda, Drools et ELK Logstash.
- Frameworks web : les surfaces d'expression embarquées dans les couches MVC et de vue côté serveur.

## Sous-sujets

- **[Frameworks web](web-frameworks/index.md)** : les frameworks web Java évaluent le langage d'expression tiré des données de requête, de l'EL unifié de JSP et JSF aux moteurs de framework SpEL, OGNL et MVEL th...

## Outils

- **Burp Suite** : Repeater et Intruder pour injecter des payloads EL, OGNL et MVEL.
- **J2EEScan** : extension Burp avec des contrôles actifs pour l'injection Java EL et OGNL.

## Références

- [Jakarta Expression Language specification](https://jakarta.ee/specifications/expression-language/)
- [Apache Commons JEXL](https://commons.apache.org/proper/commons-jexl/)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
