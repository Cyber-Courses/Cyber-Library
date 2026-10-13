---
title: "Injection de template côté serveur FreeMarker"
description: "Exploiter la SSTI FreeMarker : confirmation avec ${7*7}, exécution de commande via les built-ins Execute, new et api, et contournement de la restriction new_builtin_class_resolver."
keywords:
  - SSTI FreeMarker
  - built-in Execute
  - built-in new
  - freemarker.template.utility.Execute
  - new_builtin_class_resolver
---

# FreeMarker

FreeMarker est un moteur Java répandu (Spring, Struts, autonome). Confirmez avec `${7*7}` renvoyant `49`. Les built-ins `?` de FreeMarker en comptent plusieurs qui construisent et invoquent des objets Java, ce qui est la voie vers la RCE.

La primitive classique est l'utilitaire `Execute`, instancié avec le built-in `new` puis appelé :

```freemarker
<#assign ex = "freemarker.template.utility.Execute"?new()>${ ex("id") }
```

`"freemarker.template.utility.Execute"?new()` construit une instance d'une classe qui exécute une commande lorsqu'elle est appelée, et `ex("id")` l'exécute. Deux autres built-ins offrent des voies alternatives. `?api` expose l'API Java sous-jacente d'une valeur, permettant l'accès à un classloader :

```freemarker
${ "freemarker.template.utility.ObjectConstructor"?new()("java.lang.ProcessBuilder","id").start() }
```

`ObjectConstructor` construit un objet arbitraire à partir d'un nom de classe et d'arguments de constructeur, de sorte que construire un `ProcessBuilder` et appeler `.start()` exécute la commande. Une troisième forme atteint un `ProcessBuilder` via `?api.getClass()` et la réflexion lorsque les classes utilitaires sont bloquées.

FreeMarker a ajouté `new_builtin_class_resolver` pour restreindre les classes que `?new` peut instancier. Lorsque l'application le règle sur `TemplateClassResolver.SAFER_RESOLVER` ou `ALLOWS_NOTHING_RESOLVER`, `Execute` et `ObjectConstructor` sont rejetés et `?new` est effectivement fermé. Le built-in `?api` est de même conditionné par le paramètre `api_builtin_enabled`, désactivé par défaut. Sur une configuration durcie, testez chaque built-in : si tous sont bloqués, l'injection est limitée à la lecture des variables du modèle de données exposées. Sur une configuration par défaut ou permissive, le one-liner `Execute?new()` est une RCE immédiate.

## Outils

- tplmap, SSTImap

## Références

- Documentation Apache FreeMarker : ?new, ?api, new_builtin_class_resolver
- PortSwigger Web Security Academy: Server-side template injection
