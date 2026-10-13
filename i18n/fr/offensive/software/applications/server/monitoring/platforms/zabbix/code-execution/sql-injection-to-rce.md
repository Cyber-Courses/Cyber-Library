---
order: 4
title: "De l'injection SQL à la RCE : chaînes d'injection du frontend Zabbix"
description: "Le frontend Zabbix a connu des vulnérabilités d'injection SQL, certaines pré-authentification, dans des paramètres atteignant des requêtes de base de données de façon non sécurisée. Au-delà de l'extraction de données, une SQLi Zabbix récupère des identifiants de session et des identifiants dans la base pour obtenir une session authentifiée (souvent admin), qui atteint ensuite l'exécution de code via les fonctions de script et d'item de la plateforme."
keywords:
  - zabbix sql injection
  - jsrpc
  - pre-auth
  - session extraction
  - rce chain
---

# De l'injection SQL à la RCE

Le frontend PHP de Zabbix a livré des vulnérabilités d'injection SQL, y compris pré-authentification, où un paramètre de requête atteint une requête de base de données sans paramétrage approprié. Directement, l'injection extrait des données de la base de données Zabbix : la table `sessions` (identifiants de session actifs), la table `users` (empreintes de mots de passe et, dans certaines versions, des secrets), la table `token` (jetons d'API), et la configuration des items/macros qui contient les identifiants de supervision. La chaîne vers l'exécution de code consiste à utiliser l'injection pour récupérer l'identifiant de session ou les identifiants d'un administrateur, se connecter en tant que cet admin, puis exécuter des commandes via les fonctions [script](global-and-alert-scripts.md) ou [item](item-command-execution.md). Ainsi une SQLi Zabbix, même non authentifiée, devient une exécution de code complète sur le serveur via les propres mécanismes de la plateforme.

```bash
# the vulnerable endpoint/parameter is version-specific (e.g. historic jsrpc.php
# and various frontend parameters); fingerprint the version and match the advisory.
# the chain, once injection is confirmed:
#  1. extract an active admin session id from the sessions table:
#       UNION SELECT sessionid FROM sessions JOIN users USING(userid) WHERE users.roleid=<superadmin>
#  2. replay that session (zbx_sessionid cookie) to become admin   (see session theft)
#  3. create+execute a global script for RCE on the Zabbix server  (see global/alert scripts)
```

## Notes d'exploitation

- Le butin direct, ce sont les tables `sessions` et `token` : extraire un identifiant de session admin ou un jeton d'API via l'injection donne un contexte admin authentifié sans casser de mot de passe, rejouez-le selon [Vol de session et de jeton](../authentication/session-and-token-theft.md).
- Le point d'entrée et le paramètre dépendent de la version ; `apiinfo.version` donne le build à associer à l'avis SQLi applicable, et certaines sont pré-authentification (aucune connexion nécessaire pour injecter).
- À partir d'une session admin, la RCE passe par la voie du [script global](global-and-alert-scripts.md) sur le serveur Zabbix ; la SQLi est l'étape d'accès, les scripts sont l'étape d'exécution.
- Le même accès à la base expose aussi les identifiants de supervision dans les items/macros, l'injection est donc à la fois une primitive d'accès et de collecte d'identifiants.

## Références

- [Zabbix security advisories](https://support.zabbix.com/browse/ZBX)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
