---
order: 2
title: "Injection de template côté serveur en Java"
description: "SSTI dans les moteurs de template Java : les built-ins Execute et ObjectConstructor de FreeMarker et la chaîne de réflexion de Velocity vers Runtime.exec."
keywords:
  - SSTI Java
  - FreeMarker
  - Velocity
  - RCE par injection de template
---

# Java

Les moteurs de template Java atteignent l'exécution de code via les propres classes de la plateforme plutôt que via une couche de scripting, de sorte que l'exploitation chemine vers `java.lang.Runtime` ou `ProcessBuilder` et appelle `exec`. Les deux moteurs ici utilisent `${ ... }` pour l'interpolation, donc `${7*7}` renvoyant `49` est la sonde partagée (Velocity répond aussi à `#set($x=7*7)$x`).

La différence entre eux est le gadget : FreeMarker fournit des built-ins utilitaires (`Execute`, `ObjectConstructor`, `new`) qui construisent et exécutent des objets directement, tandis que Velocity n'a pas de tel assistant et s'exploite en réfléchissant depuis une classe disponible vers `Runtime`.

## Moteurs

- **[FreeMarker](freemarker.md)** : les built-ins `Execute`, `new` et `api`, et le verrou `new_builtin_class_resolver`.
- **[Velocity](velocity.md)** : réflexion depuis `$class.inspect(...)` vers `Runtime.getRuntime().exec`.

## Références

- Documentation Apache FreeMarker et Apache Velocity
- PortSwigger Web Security Academy: Server-side template injection
