---
order: 2
title: "De l'injection PostgreSQL à l'exécution de commandes"
description: "Les deux chemins d'une injection PostgreSQL privilégiée vers l'exécution de commandes OS : COPY ... FROM PROGRAM, et des fonctions dans un langage procédural ou C non sécurisé."
keywords:
  - exécution de commandes PostgreSQL
  - COPY FROM PROGRAM
  - plpythonu
  - plperlu
  - RCE
---

# Exécution de commandes

PostgreSQL atteint l'exécution de commandes OS plus directement que MySQL car les primitives sont intégrées au serveur. Les deux chemins nécessitent un rôle privilégié, et tous deux nécessitent habituellement une requête empilée afin qu'une instruction propre puisse s'exécuter.

Le chemin le plus simple est `COPY ... FROM PROGRAM`, qui exécute une commande shell et capture sa sortie dans une table. Il nécessite un superutilisateur ou le rôle `pg_execute_server_program` et existe depuis PostgreSQL 9.3.

Le second chemin définit une fonction dans un langage non sécurisé. Créer des fonctions en `plpythonu`, `plperlu` ou C nécessite un superutilisateur, et une fois créée, la fonction exécute du code natif, de sorte qu'un wrapper autour de `subprocess` ou `system()` donne une exécution de commandes appelable depuis le SQL.

## Pages

- **[COPY FROM PROGRAM](copy-from-program.md)** : exécuter une commande et lire sa sortie via une table.
- **[Fonction en langage non sécurisé](language-function.md)** : définir une fonction `plpythonu`, `plperlu` ou C qui exécute des commandes.

## Outils

- **sqlmap** : automatise l'exécution de commandes avec `--os-shell` via COPY FROM PROGRAM.
- **Metasploit Framework** : module `postgres_payload` pour l'exécution de commandes sur les rôles privilégiés.

## Références

- PostgreSQL Documentation: COPY ... FROM PROGRAM, CREATE FUNCTION, procedural languages
- OWASP Testing Guide: Testing for SQL Injection
