---
order: 6
title: "Énumération de l'instance et du SID dans l'injection Oracle"
description: "Identifier l'instance, le SID et le nom de service Oracle depuis une injection SQL, et comment les informations du listener la complètent pour le ciblage."
keywords:
  - SID Oracle
  - nom d'instance
  - nom de service
  - v$instance
  - global_name
---

# Énumération du listener et du SID

Connaître l'identité de l'instance (SID, nom de service, nom global, hôte) affine les étapes ultérieures et est nécessaire pour toute connexion directe à la base de données. Une injection SQL en lit l'essentiel depuis le dictionnaire de données.

L'identité de l'instance et du service provient des vues `V$` et de `SYS_CONTEXT` :

```sql
' UNION SELECT instance_name,host_name,version FROM v$instance-- 
' UNION SELECT SYS_CONTEXT('USERENV','DB_NAME'),SYS_CONTEXT('USERENV','INSTANCE_NAME'),SYS_CONTEXT('USERENV','SERVICE_NAME') FROM dual-- 
' UNION SELECT global_name,NULL,NULL FROM global_name-- 
```

`v$instance` donne le nom de l'instance et l'hôte, `global_name` le nom global de la base de données, et `SYS_CONTEXT` le nom de la base, le nom de l'instance et le nom de service. Ces éléments identifient le SID/service nécessaire pour connecter un client comme SQLcl ou un outil comme ODAT directement au listener.

Au-delà de l'injection, le listener TNS d'Oracle sur le TCP 1521 est lui-même une cible : il répond aux requêtes de service et de version, et là où le SID ou le nom de service est inconnu, il peut être brute-forcé contre le listener. Un listener hérité, non authentifié, peut divulguer son statut directement. Ce sont des interactions réseau contre le listener plutôt que de l'injection SQL, mais elles se marient avec l'identité en bande ci-dessus : l'injection confirme le SID et le nom de service, et le listener est ensuite la voie pour une session directe à plus haut débit une fois des identifiants ou un hash récupérés.

## Outils

- **ODAT** (module sidguesser) : brute-force le SID et le nom de service contre le listener TNS.
- **Nmap** (scripts NSE `oracle-sid-brute`, `oracle-tns-version`) : interroge le listener sur le TCP 1521.
- **sqlplus** (ou SQLcl) : se connecte directement une fois le SID et les identifiants connus.

## Références

- Oracle Database Reference : V$INSTANCE, GLOBAL_NAME, SYS_CONTEXT
- OWASP Testing Guide : Testing for SQL Injection
