---
order: 1
title: "Désérialisation non sécurisée : transformer des objets contrôlés par l'attaquant en exécution de code"
description: "Comment les sinks de désérialisation deviennent des RCE : callbacks du moteur qui se déclenchent pendant la reconstruction d'objets, chaînes de gadgets, et points d'entrée des données sérialisées contrôlées par l'attaquant, organisé par langage."
keywords:
  - désérialisation non sécurisée
  - injection d'objet
  - chaîne de gadgets
  - pickle
  - ysoserial
  - chaîne POP
---

# Désérialisation non sécurisée

La sérialisation transforme un objet en mémoire en flux d'octets ; la désérialisation le reconstruit. La vulnérabilité tient au fait que la plupart des moteurs exécutent des **callbacks définis par le moteur pendant la reconstruction** (méthodes magiques, reducers, hooks de relecture), de sorte qu'un désérialiseur recevant des octets contrôlés par l'attaquant ne produit pas seulement des données, il *exécute un comportement*. Chaîner ces callbacks à travers des classes déjà chargées dans l'application atteint l'exécution de commandes, l'écriture de fichiers ou la SSRF. L'impact est généralement une **exécution de code à distance**.

## Le modèle commun

Chaque variante de langage suit le même schéma en trois parties :

1. **Un sink** qui désérialise une entrée non fiable : `unserialize()`, `pickle.loads()`, `ObjectInputStream.readObject()`, `Marshal.load()`, `BinaryFormatter.Deserialize()`, `node-serialize.unserialize()`.
2. **Un déclencheur** : un callback que le moteur exécute automatiquement sur l'objet reconstruit (`__wakeup`/`__destruct`, `__reduce__`, `readObject`, `init_with`, une IIFE), ou un type que le désérialiseur est chargé d'instancier.
3. **Une chaîne de gadgets** : une suite de méthodes sur des classes disponibles dans le code de la cible et ses dépendances qui, une fois atteinte via le déclencheur, réalise l'action de l'attaquant. La chaîne est assemblée à partir de ce qui est sur le classpath, ce qui explique que les outils fournissent des chaînes prêtes à l'emploi par framework.

## Trouver le sink

- Reconnaître les formats sérialisés sur le réseau : PHP `O:4:"User":...`, Java base64 commençant par `rO0` (hex `AC ED 00 05`), opcodes de pickle Python, .NET `AAEAAAD/////`, Marshal Ruby `\x04\x08`.
- Chercher dans les cookies, les champs cachés, les en-têtes `Authorization`/personnalisés, les caches, les files de messages et les fonctionnalités d'import/téléversement.
- Confirmer en altérant un octet et en guettant une erreur spécifique à la désérialisation, puis passer à un objet contrôlé.

## Langages

- **[Injection d'objet PHP](php-object-injection.md)** : `unserialize()`, chaînes POP via méthodes magiques, et atteindre le sink sans appel explicite via `phar://`.
- **[Pickle Python](python-pickle.md)** : `__reduce__` comme primitive de RCE directe, plus `yaml.load` et jsonpickle.
- **[Désérialisation Java](java-deserialization.md)** : `readObject`, chaînes de gadgets ysoserial et points d'entrée courants.
- **[Désérialisation Ruby](ruby-deserialization.md)** : chaînes de gadgets universelles `Marshal.load` et `YAML.load`.
- **[Désérialisation .NET](dotnet-deserialization.md)** : `BinaryFormatter`, `TypeNameHandling` et ViewState.
- **[Désérialisation Node.js](nodejs-deserialization.md)** : `node-serialize` et bibliothèques de réanimation de fonctions.

## Outils

- **ysoserial** (Java), **ysoserial.net**, **PHPGGC** (chaînes de gadgets PHP), **marshalsec** (Java/JVM)

## Références

- PortSwigger Web Security Academy: Insecure deserialization
- OWASP: Deserialization Cheat Sheet
