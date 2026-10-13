---
order: 1
title: "Injection d'objet PHP : chaînes POP via méthodes magiques de unserialize()"
description: "Exploiter unserialize() de PHP sur une entrée de l'attaquant : forger des objets sérialisés, pousser des méthodes magiques (__wakeup, __destruct, __toString) dans des chaînes de programmation orientée propriétés, et atteindre le sink via phar."
keywords:
  - injection d'objet PHP
  - unserialize
  - chaîne POP
  - __wakeup
  - __destruct
  - PHPGGC
---

# Injection d'objet PHP

L'injection d'objet PHP survient lorsque `unserialize()` est appelée sur des données contrôlées par l'attaquant. PHP reconstruit le graphe d'objets décrit par la chaîne, y compris sa classe et les valeurs de ses propriétés, et exécute des **méthodes magiques** pendant et après la reconstruction. Un attaquant qui contrôle la chaîne sérialisée contrôle quelles classes sont instanciées et avec quelles propriétés, puis chevauche les méthodes magiques jusqu'à une **chaîne de programmation orientée propriétés (POP)**.

## Le format sérialisé

La sérialisation PHP est lisible par un humain et forgeable à la main :

```
O:4:"User":2:{s:4:"name";s:5:"admin";s:7:"isAdmin";b:1;}
```

`O:4:"User"` est un objet d'une classe au nom de 4 caractères `User` avec 2 propriétés. Les types sont explicites (`s` string, `i` int, `b` bool, `a` array, `O` object), de sorte que vous pouvez forger n'importe quel objet de n'importe quelle classe que l'application peut autoloader, avec n'importe quelles valeurs de propriétés, y compris des objets imbriqués.

## Les déclencheurs

PHP exécute ces méthodes magiques sur un objet reconstruit sans que l'application les appelle :

- `__wakeup()` : s'exécute immédiatement à `unserialize()`.
- `__destruct()` : s'exécute lorsque l'objet est collecté par le ramasse-miettes en fin de requête.
- `__toString()` : s'exécute lorsque l'objet est utilisé dans un contexte de chaîne.
- `__call()`, `__get()`, `__set()` : s'exécutent à l'accès à des méthodes/propriétés manquantes.

N'importe laquelle de celles-ci sur une classe chargeable est un point d'entrée. L'attaque n'a pas besoin que la classe soit manifestement dangereuse ; elle a besoin d'une méthode qui, lorsqu'elle s'exécute avec des propriétés choisies par l'attaquant, fait quelque chose d'utile, ou qui appelle la méthode d'un autre objet, prolongeant la chaîne.

## Chaînes POP

Une chaîne POP enchaîne des méthodes à travers des classes qui existent déjà dans l'application et ses dépendances. Une forme représentative : le `__destruct()` d'un objet d'entrée appelle `$this->handler->close()` ; l'attaquant fixe `handler` à un objet dont le `close()` écrit `$this->data` dans `$this->path` ; l'attaquant fixe `path` à un fichier du webroot et `data` à un payload PHP. La reconstruction plus `__destruct` devient une écriture de fichier arbitraire.

Parce que les chaînes dépendent des bibliothèques exactes présentes, utilisez **PHPGGC** pour générer des payloads pour les frameworks connus (Laravel, Symfony, Monolog, Guzzle, Doctrine, WordPress) :

```bash
phpggc Monolog/RCE1 system 'id'            # print a payload
phpggc -b Monolog/RCE1 system 'id'         # base64-encoded
phpggc Guzzle/RCE1 system 'id'
```

Alimentez le sink (cookie, paramètre, en-tête) avec la sortie exactement comme l'application l'attend.

## Atteindre unserialize sans appel explicite

Un sink ne requiert pas un `unserialize()` littéral sur votre entrée. PHP désérialise les métadonnées d'une archive Phar lorsqu'une opération de fichier touche un chemin `phar://`, de sorte que des fonctions de fichier agissant sur un chemin influencé par l'attaquant deviennent des sinks de désérialisation. Voir [désérialisation phar](../file-wrappers-and-stream-handlers/phar-deserialization.md) pour construire l'archive et la déclencher.

## Notes d'exploitation

- `__wakeup()` était historiquement contournable (un bug de non-correspondance du nombre d'objets) pour déjouer des gardes ; sur le PHP moderne, fiez-vous à `__destruct()` et aux autres méthodes magiques.
- Les noms de propriétés privées et protégées se sérialisent avec des préfixes d'octet nul (`\0ClassName\0prop` / `\0*\0prop`) ; préservez-les exactement lors d'une forge à la main.
- Énumérez les classes chargeables et leurs méthodes magiques à partir du source ou de chemins fuités ; la chaîne ne vaut que par les gadgets présents sur le disque.

## Outils

- **PHPGGC** : chaînes de gadgets sélectionnées pour les frameworks courants.

## Références

- PortSwigger Web Security Academy: Insecure deserialization (PHP)
- PHP manual: Object serialization, magic methods
