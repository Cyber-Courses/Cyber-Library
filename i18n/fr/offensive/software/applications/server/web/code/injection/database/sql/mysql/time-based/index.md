---
order: 8
title: "Injection SQL aveugle basée sur le temps dans MySQL"
description: "Déduire des données MySQL à partir de délais de réponse conditionnels avec SLEEP et BENCHMARK quand la réponse est par ailleurs identique pour le vrai et le faux."
keywords:
  - aveugle basée sur le temps
  - injection SLEEP
  - BENCHMARK
  - délai conditionnel
  - timing aveugle MySQL
---

# Basée sur le temps

Lorsque les conditions vraies et fausses produisent la même réponse, l'inférence se rabat sur le temps. Un payload qui ne retarde la réponse que lorsqu'une condition est vraie transforme la latence de la réponse en signal oui/non : une réponse lente signifie vrai, une réponse immédiate signifie faux.

MySQL temporise avec `SLEEP(seconds)`, qui met en pause l'évaluation de la ligne courante, et avec `BENCHMARK(count, expr)`, qui sollicite le CPU en répétant une expression. Le délai doit être rendu conditionnel pour qu'il ne se déclenche que sur un test vrai, ce qui porte un bit d'information par requête.

Le détail de dialecte essentiel est que le délai a besoin d'un contexte de ligne ou d'un contexte d'expression. `SELECT SLEEP(5) WHERE <test>` est une erreur de syntaxe sous MySQL car il n'autorise un `SELECT` sans `FROM` que pour des constantes, et un `WHERE` sans `FROM` est invalide (contrairement à PostgreSQL et SQLite). Le délai est donc placé à l'intérieur d'`IF()` ou rattaché à une source de lignes. Les pages ci-dessous montrent les deux formes correctes.

Le timing est plus bruyant qu'un oracle booléen car la gigue réseau et la charge du serveur influent sur la latence, les délais sont donc maintenus à plusieurs secondes et confirmés par une répétition avant qu'un bit ne soit considéré comme fiable.

## Pages

- **[SLEEP](sleep.md)** : extraction conditionnelle `IF(..., SLEEP(n), 0)`.
- **[BENCHMARK](benchmark.md)** : délai basé sur le CPU quand `SLEEP` est filtré.

## Outils

- **sqlmap** : extraction aveugle temporelle automatisée avec SLEEP et BENCHMARK.
- **ghauri** : alternative rapide avec une forte évasion de WAF.

## Références

- MySQL Reference Manual : SLEEP, BENCHMARK, IF
- PortSwigger Web Security Academy : Blind SQL injection
