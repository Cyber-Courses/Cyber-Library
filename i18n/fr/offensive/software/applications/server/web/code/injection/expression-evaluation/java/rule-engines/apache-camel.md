---
title: "Injection Apache Camel : exécution de code via les langages d'expression de route"
description: "Les routes Camel évaluent des expressions dans les langages Simple, OGNL ou SpEL, et une entrée non fiable intégrée dans une expression de route atteint l'invocation de bean et de méthode et, à travers elles, le runtime de la JVM."
keywords:
  - Apache Camel
  - langage Camel Simple
  - Camel OGNL
  - Camel SpEL
  - injection d'expression de route
---

# Apache Camel

Apache Camel est un framework d'intégration bâti autour de routes qui déplacent et transforment des messages. Les routes utilisent des expressions partout : prédicats pour le routage basé sur le contenu, URIs d'endpoints dynamiques, setters d'en-tête et de corps, et chaînes templatisées. Camel prend en charge plusieurs langages d'expression pour cela, et celui qui est en jeu décide jusqu'où une expression injectée atteint. L'exposition est toute expression de route construite à partir du contenu d'un message ou d'une configuration qu'un attaquant peut influencer, et le plafond dépend du langage que la route utilise.

## Le langage Simple

Le langage léger par défaut de Camel est Simple (`${...}`). Il a commencé comme un langage de templating de chaînes et de prédicats mais prend en charge l'invocation de bean et de méthode, ce qui en fait une surface d'exécution de code plutôt que de formatage. Une expression Simple injectée peut appeler une méthode sur un bean atteignable ou construire et invoquer via le langage bean :

```
${bean:myBean?method=process(payload)}
```

Là où le corps du message ou un en-tête est interpolé dans une expression Simple que la route évalue ensuite, l'attaquant contrôle la cible de l'appel de méthode et les arguments parmi les beans que le registry expose.

## Composants OGNL et SpEL

Lorsqu'une route utilise les composants de langage OGNL ou SpEL (`.ognl(...)`, `.spel(...)`, ou les endpoints `language:ognl` / `language:spel`), l'expression injectée est de l'OGNL ou du Spring Expression Language complet. Les deux résolvent les noms de classes et invoquent des méthodes arbitraires, donc ils atteignent le runtime directement. OGNL :

```
(#rt=@java.lang.Runtime@getRuntime()).exec('id')
```

SpEL :

```
T(java.lang.Runtime).getRuntime().exec('id')
```

Relire la sortie enveloppe le flux `Process` renvoyé dans la même grammaire d'expression (un `InputStreamReader`/`Scanner` sur `getInputStream()`), faisant atterrir le résultat dans le message que la route produit.

## Les fonctionnalités de shell nécessitent un vecteur d'arguments

`Runtime.exec(String)` tokenise sur les espaces sans shell, donc `$(...)`, les pipes et les redirections ne s'étendent pas. Construisez un vecteur explicite pour le comportement de shell, montré en SpEL :

```
T(java.lang.Runtime).getRuntime().exec(new String[]{'/bin/bash','-c','id > /tmp/o 2>&1'})
```

et de manière équivalente en OGNL avec `new java.lang.String[]{...}`. Sous Windows utilisez `{'cmd.exe','/c','whoami'}`. Parce que la portée dépend du langage, confirmez quel langage d'expression la route évalue avant de vous engager : un sink Simple uniquement vous limite aux beans exposés et à leurs méthodes, tandis qu'un sink OGNL ou SpEL donne la résolution de classe directe et le runtime complet.

## Outils

- **Burp Suite** : Repeater pour injecter dans les expressions de route Simple, OGNL ou SpEL.
- **J2EEScan** : extension Burp signalant l'injection d'expression OGNL et SpEL.

## Références

- [Apache Camel: Simple language](https://camel.apache.org/components/latest/languages/simple-language.html)
- [Apache Camel: SpEL language](https://camel.apache.org/components/latest/languages/spel-language.html)
- [Apache Camel: OGNL language](https://camel.apache.org/components/latest/languages/ognl-language.html)
