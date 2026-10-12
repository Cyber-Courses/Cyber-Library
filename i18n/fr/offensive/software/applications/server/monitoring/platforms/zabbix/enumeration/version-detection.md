---
title: "Détection de version : lire la version de Zabbix"
description: "La version de Zabbix est divulguée sans authentification via la méthode apiinfo.version de l'API et le pied de page de la page de connexion, et c'est le fait qui décide quels exploits s'appliquent. Faire correspondre la version sépare les instances corrigées de celles exposées aux chaînes connues d'injection SQL et d'exécution de code à distance."
keywords:
  - zabbix version
  - apiinfo.version
  - footer
  - fingerprint
  - exploit matching
---

# Détection de version

La version de Zabbix est à la fois facile à obtenir et décisive, car les chaînes d'injection SQL et d'exécution de code à distance sont spécifiques à la version. L'API l'expose sans authentification via `apiinfo.version` (l'une des rares méthodes non authentifiées), et l'interface web l'affiche dans la chaîne « Zabbix x.y.z » de la page de connexion et du pied de page. La lire en premier vous indique si l'instance est à portée d'un exploit pré-authentification connu, et quels comportements d'authentification et d'exécution de code attendre (les valeurs par défaut, la forme de l'API et le traitement des scripts ont changé d'une version majeure à l'autre).

```bash
Z=https://<target>/zabbix/api_jsonrpc.php
# unauthenticated version via the API
curl -sk $Z -H 'Content-Type: application/json-rpc' \
  -d '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'
# version from the web UI footer/login page
curl -sk https://<target>/zabbix/index.php | grep -ioE 'Zabbix [0-9]+\.[0-9]+\.[0-9]+'
```

## Notes d'exploitation

- `apiinfo.version` répond sans identifiants, donc la version est gratuite ; elle se mappe directement aux [vulnérabilités serveur connues](../code-execution/known-server-exploits.md) applicables et aux chaînes SQLi.
- Les versions majeures diffèrent dans leurs valeurs par défaut et leurs fonctionnalités (accès invité, jetons d'API, modèle d'exécution de scripts), de sorte que la version sélectionne aussi quelles techniques d'authentification et d'exécution de code s'appliquent.
- La chaîne du pied de page corrobore le résultat de l'API et fonctionne lorsque le chemin de l'API est filtré ; les deux sont pré-authentification.
- Injectez la version dans le choix des exploits, puis énumérez les [utilisateurs](user.md) et les [hôtes](host.md) une fois l'accès obtenu.

## Références

- [Zabbix API : apiinfo.version](https://www.zabbix.com/documentation/current/en/manual/api/reference/apiinfo/version)
- [HackTricks : Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
