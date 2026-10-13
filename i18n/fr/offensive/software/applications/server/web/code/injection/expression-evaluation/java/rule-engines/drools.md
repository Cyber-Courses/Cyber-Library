---
title: "Injection Drools : exécution de code via le dialecte MVEL"
description: "Les conditions et conséquences de règles Drools s'évaluent dans le dialecte MVEL, qui a un accès Java complet, donc une entrée non fiable intégrée dans une expression ou une conséquence Drools exécute du MVEL et atteint le runtime de la JVM."
keywords:
  - Drools
  - injection Drools
  - dialecte MVEL
  - DRL
  - exécution de code JVM
---

# Drools

Drools est un moteur de règles métier. Les règles sont écrites en DRL (Drools Rule Language), où chaque règle a un membre gauche de conditions sur les faits de la working memory et un membre droit de conséquence qui s'exécute lorsque la règle se déclenche. Les expressions dans les conditions, et le code dans les conséquences, sont évalués par un dialecte, et le dialecte d'expression par défaut de Drools est MVEL. MVEL est un langage d'expression JVM complet avec construction d'objets, invocation de méthodes et réflexion, donc le vecteur d'injection ici est MVEL sous Drools : tout ce qui construit du texte DRL ou une expression MVEL à partir d'une entrée non fiable exécute du MVEL de l'attaquant, et MVEL atteint le runtime.

## Pattern vulnérable

L'exposition apparaît partout où le texte des règles est assemblé dynamiquement plutôt que chargé depuis des ressources fixes, par exemple en construisant du DRL à partir d'une condition ou d'un seuil fourni par l'utilisateur et en le compilant :

```java
String drl =
  "rule \"dynamic\"\n" +
  "when\n" +
  "  $o : Order( " + userCondition + " )\n" +
  "then\n" +
  "  " + userAction + "\n" +
  "end";
// drl compiled into a KieBase and fired
```

À la fois la condition tissée et la conséquence sont du MVEL. Les applications qui laissent les utilisateurs rédiger des règles, ou qui passent une entrée utilisateur dans un `eval` MVEL ou une cellule de table de décision, exposent le même dialecte.

## Atteindre le runtime

MVEL résout les noms de classes et appelle des méthodes, donc une conséquence (ou une expression MVEL injectée) obtient le runtime et exécute une commande :

```
Runtime.getRuntime().exec("id");
```

MVEL prend aussi en charge la construction explicite, donc `ProcessBuilder` fonctionne directement :

```
new java.lang.ProcessBuilder("id").start();
```

Pour capturer la sortie, enveloppez le flux de processus dans la conséquence, en construisant un `java.util.Scanner` sur `getInputStream()` et en lisant un token, puis liez-le à un champ de fait ou à un objet inséré que l'application lit plus tard.

## Les fonctionnalités de shell nécessitent un vecteur d'arguments

`Runtime.exec(String)` tokenise sur les espaces et s'exécute sans shell, donc `$(...)`, les pipes et les redirections restent littéraux. Pour un comportement de shell, construisez le vecteur d'arguments et laissez bash l'interpréter :

```
new java.lang.ProcessBuilder(new String[]{"/bin/bash","-c","id > /tmp/o 2>&1"}).start();
```

Sous Windows utilisez `new String[]{"cmd.exe","/c","whoami"}`. Si l'application a épinglé le dialecte à `java` au lieu de `mvel`, la conséquence est du Java compilé plutôt que du MVEL mais atteint la même API `Runtime`/`ProcessBuilder`, donc le sink de commande est inchangé.

## Outils

- **Burp Suite** : Repeater pour délivrer des payloads de condition et de conséquence MVEL.
- Payloads MVEL manuels atteignant Runtime et ProcessBuilder.

## Références

- [Drools documentation](https://docs.drools.org/)
- [MVEL language guide](http://mvel.documentnode.com/)
- [Drools: rule language reference (DRL)](https://docs.drools.org/latest/drools-docs/drools/language-reference/index.html)
