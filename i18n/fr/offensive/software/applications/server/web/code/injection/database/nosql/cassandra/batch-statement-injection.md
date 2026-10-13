---
title: "Injection d'instruction batch : introduire clandestinement des écritures dans un BATCH CQL"
description: "Injecter dans un bloc BEGIN BATCH ... APPLY BATCH, ou en former un, permet à un attaquant de regrouper des écritures INSERT, UPDATE et DELETE supplémentaires dans une seule instruction Cassandra."
keywords:
  - injection de batch CQL
  - BEGIN BATCH
  - APPLY BATCH
  - écritures clandestines
  - Cassandra
---

# Injection d'instruction batch

CQL regroupe plusieurs opérations d'écriture en une seule unité atomique avec `BEGIN BATCH ... APPLY BATCH`. Lorsqu'une entrée utilisateur est concaténée dans une instruction qui est, ou peut être transformée en, un batch, un attaquant introduit clandestinement des écritures `INSERT`, `UPDATE` ou `DELETE` supplémentaires aux côtés de celle que l'application avait prévue.

## Pourquoi les batchs comptent

La plupart des pilotes Cassandra rejettent plusieurs instructions séparées par des points-virgules dans un même `execute`, donc l'astuce classique des requêtes empilées en SQL (`; DROP ...`) échoue généralement. Un **batch** est la manière sanctionnée d'exécuter plusieurs écritures en une seule instruction, et c'est une seule instruction du point de vue du pilote. Si l'injection peut atteindre le corps d'un batch, ou transformer une écriture unique en batch, des écritures supplémentaires s'y greffent sans nécessiter d'empilement d'instructions.

## Injecter dans un batch existant

Les applications construisent parfois un batch en concaténant des chaînes par élément dans une boucle :

```python
cql = "BEGIN BATCH\n"
for item in items:
    cql += "INSERT INTO cart (user, sku) VALUES ('%s','%s');\n" % (user, item)
cql += "APPLY BATCH;"
session.execute(cql)
```

Une valeur `item` de :

```sql
x'); INSERT INTO users (id, role) VALUES ('attacker','admin') //
```

ferme l'`INSERT` courant, en ajoute un choisi, et se termine par un commentaire de ligne `//` (CQL accepte aussi `--` et `/* */`) pour avaler le `')` que le gabarit ajoute encore. Elle se développe en :

```sql
BEGIN BATCH
INSERT INTO cart (user, sku) VALUES ('alice','x'); INSERT INTO users (id,role) VALUES ('attacker','admin') //');
APPLY BATCH;
```

Le commentaire écarte le `');` restant, l'`INSERT` clandestin écrit une ligne admin, et l'`APPLY BATCH` du gabarit termine le bloc. Parce que les instructions dans un batch sont séparées par `;` et que le bloc entier est une seule instruction du pilote, cela fonctionne là où les requêtes empilées autonomes échouent.

## Former un batch à partir d'une écriture unique

Lorsque le point d'injection est un unique `INSERT` ou `UPDATE` plutôt qu'un batch existant, l'injection peut l'envelopper. Étant donné :

```python
q = "INSERT INTO audit (actor, note) VALUES ('%s','%s')" % (actor, note)
```

une valeur forgée peut fermer la première écriture et en ajouter une seconde, à condition que le pilote accepte la forme combinée en tant que batch. Préfixer l'instruction est rarement atteignable, donc la cible pratique est une application qui utilise déjà `BEGIN BATCH`, ou un `UPDATE` dont le `SET` ou le `WHERE` est injectable.

## Écraser et supprimer

À l'intérieur d'un batch atteignable, toute la surface d'écriture est disponible :

```sql
'); UPDATE users SET role='admin' WHERE id='victim'
'); DELETE FROM sessions WHERE id='victim'
'); INSERT INTO users (id,password) VALUES ('bd','...')
```

Un `UPDATE` en CQL est un upsert, donc un `UPDATE` sur une clé primaire inexistante crée la ligne. Cela fait de l'`UPDATE` un moyen fiable à la fois de modifier un enregistrement existant et d'en planter un nouveau. Les batchs acceptent aussi une clause `USING TIMESTAMP`, qui peut être fixée à une valeur élevée pour que l'écriture clandestine l'emporte dans la résolution de conflit face à des écritures légitimes ultérieures :

```sql
'); UPDATE users USING TIMESTAMP 99999999999999 SET role='admin' WHERE id='victim'
```

## Conditions et limites

Les batchs journalisés s'étendent sur plusieurs partitions mais ne sont pas des transactions isolées, et les conditions de transaction légère (`IF`) s'appliquent par instruction. Pour l'injection, le point clé est plus simple : tout ce que le rôle de base de données de l'application est autorisé à écrire, une instruction de batch clandestine peut aussi l'écrire. Cartographier les privilèges de ce rôle, via le sondage de schéma [par erreur](cql/error-based.md), indique quelles tables sont atteignables avant de forger les écritures supplémentaires.

## Outils

- **cqlsh** : confirmer que les écritures INSERT, UPDATE ou DELETE clandestines atterrissent dans les tables cibles.
- **Burp Repeater** : forger des requêtes portant le payload d'évasion de batch.

## Références

- [Apache Cassandra: BATCH](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/dml.html#batch_statement)
- [Apache Cassandra: INSERT, UPDATE, DELETE](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/dml.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
