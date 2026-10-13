---
order: 1
title: "Inférence en aveugle : extraction booléenne depuis des requêtes CQL"
description: "Lorsqu'une requête Cassandra ne renvoie aucune donnée, un attaquant la lit un fait à la fois en forgeant des prédicats dont la vérité change le fait que des lignes soient renvoyées ou non."
keywords:
  - injection CQL en aveugle
  - inférence booléenne
  - Cassandra
  - ALLOW FILTERING
  - extraction de données
---

# Inférence en aveugle

Lorsqu'une requête CQL injectable ne renvoie aucune donnée visible, seulement une différence de comportement entre une condition vraie et une condition fausse, les données sont récupérées un bit à la fois. L'attaquant fournit un prédicat dont il veut apprendre la vérité et observe si la réponse change.

## Pourquoi booléen, et non temporel

CQL n'a pas de fonction sleep, benchmark ou délibérément coûteuse, donc il n'y a pas de canal temporel sur lequel s'appuyer. La seule primitive d'inférence générale est **booléenne** : le fait que la requête renvoie des lignes, et la façon dont l'application y réagit. Le signal observable est généralement l'un des suivants :

- Une page qui s'affiche différemment lorsqu'au moins une ligne correspond par rapport à aucune.
- Une connexion, une recherche ou un basculement qui réussit ou échoue selon le nombre de lignes.
- Une différence de taille ou de statut entre les deux réponses.

## La paire vrai/faux

Partez d'une condition connue pour être vraie et d'une connue pour être fausse, et confirmez que les deux produisent des réponses distinguables. Étant donné un point d'injection qui filtre sur une clé de partition, un commentaire de bloc écarte la queue du gabarit et `ALLOW FILTERING` rend le prédicat ajouté acceptable :

```sql
existing' AND role = 'admin' ALLOW FILTERING /*
existing' AND role = 'nonexistent_zzzz' ALLOW FILTERING /*
```

Si le premier renvoie des lignes et le second non, le canal fonctionne et la cible possède au moins une ligne admin.

## Extraire une valeur caractère par caractère

CQL prend en charge les comparaisons de plage et les tests de sous-chaîne sur le texte, ce qui suffit pour parcourir une valeur. `token()` neutralise l'exigence de clé de partition pour que le scan atteigne la ligne cible :

```sql
' AND token(id) >= token('') AND username = 'admin' AND password > 'm' ALLOW FILTERING /*
```

Une recherche binaire sur la borne de comparaison (`>`, `>=`, `<`) affine chaque caractère. Lorsque la colonne stockée est du texte et prend en charge la correspondance de préfixe, `LIKE` (sur les tables disposant d'un index SASI approprié) ou une séquence de bornes de plage resserre chaque position :

```sql
' AND username = 'admin' AND password >= 'ma' AND password < 'mb' ALLOW FILTERING /*
```

Chaque sonde répond à une comparaison ; l'état lignes-ou-pas-de-lignes de la réponse est l'oracle. Répétez, en faisant avancer le préfixe connu, jusqu'à ce que la valeur entière soit reconstruite.

## Confirmer la présence de lignes et de colonnes

L'inférence booléenne confirme aussi des faits structurels sans les lire directement. Tester si une valeur existe quelque part dans une colonne scannée :

```sql
' AND email = 'target@example.com' ALLOW FILTERING /*
```

Une réponse avec lignes renvoyées confirme que l'enregistrement existe. Itérer sur des valeurs candidates énumère l'appartenance, lentement mais sûrement, sur un canal qui ne renvoie rien d'autre que la présence.

## Automatiser

Chaque caractère coûte généralement une poignée de requêtes sous recherche binaire. Scripter la boucle de sondage, soumettre le payload, classer la réponse comme vraie ou fausse, et faire avancer la recherche, rend l'extraction d'un secret multi-caractères praticable. Le texte d'erreur, lorsqu'il fuit, donne un canal plus rapide ; voir [Par erreur](error-based.md).

## Outils

- **cqlsh** : vérifier la paire de prédicats vrai/faux avant de scripter l'inférence.
- **Burp Intruder** : automatiser la boucle de sondage booléen par caractère.

## Références

- [Apache Cassandra: SELECT and operators](https://cassandra.apache.org/doc/latest/cassandra/developing/cql/dml.html#select-statement)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
