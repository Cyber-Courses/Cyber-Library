---
order: 2
title: "Moteurs de règles Java : injection dans les expressions de routage, de workflow et de pipeline"
description: "Les produits de règles, de routage, de workflow et de pipeline de logs évaluent des expressions et des conséquences sur leurs entrées, et le texte d'expression contrôlé par l'attaquant atteint le langage sous-jacent et le runtime de la JVM."
keywords:
  - injection moteur de règles
  - Drools MVEL
  - langage Simple de Camel
  - Camunda JUEL
  - filtre ruby Logstash
---

# Moteurs de règles

Les produits de règles, de routage, de workflow et de pipeline permettent aux opérateurs d'exprimer une logique sous forme de données : une condition de route Camel, une conséquence de règle Drools, une expression de processus Camunda, un filtre Logstash. Chacun embarque sa propre couche d'expression ou de script, et chacun évalue cette couche face aux messages ou aux événements qui le traversent. Lorsqu'une partie quelconque de cette expression est construite à partir d'une entrée non fiable, l'attaquant contrôle la logique exécutée par le moteur, et sur ces moteurs cette logique atteint un langage complet avec accès à l'hôte.

Chaque moteur de ce groupe atteint l'exécution de code, mais au travers d'un langage différent :

- [Apache Camel](apache-camel.md) : le langage Simple (`${...}`) invoque des beans et des méthodes, et les routes utilisant les composants OGNL ou SpEL atteignent les appels de méthode et le runtime.
- [Camunda](camunda.md) : BPMN utilise l'Unified EL (JUEL) `${...}` lié à des méthodes Java, et les tâches de script exécutent directement du Groovy ou du JavaScript.
- [Drools](drools.md) : les conditions et les conséquences de règle utilisent le dialecte MVEL, qui dispose d'un accès Java complet.
- [ELK Logstash](elk-logstash.md) : le filtre `ruby` du pipeline exécute du Ruby arbitraire fourni dans son option `code`.

Les moteurs JVM présents ici partagent la nuance shell de `Runtime.exec(String)` : la forme à chaîne unique découpe en tokens sur les espaces sans shell, donc les fonctionnalités de shell nécessitent un vecteur explicite `String[]{"/bin/bash","-c","..."}` au travers de `ProcessBuilder`. La page Logstash couvre l'équivalent Ruby.

## Outils

- **Burp Suite** : Repeater et Intruder pour injecter dans les expressions de route, de règle et de pipeline.
- **J2EEScan** : extension Burp avec des contrôles pour l'injection Java EL et OGNL.

## Références

- [Apache Camel: expression languages](https://camel.apache.org/components/latest/languages/index.html)
- [Drools documentation](https://docs.drools.org/)
- [Camunda: expression language](https://docs.camunda.org/manual/latest/user-guide/process-engine/expression-language/)
