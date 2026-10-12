---
title: "PRTG Network Monitor : attaquer le serveur de supervision"
order: 3
description: "PRTG est une plateforme de supervision Windows dotée d'une interface web et d'un compte prtgadmin par défaut. Sa faiblesse caractéristique est la fonctionnalité de notification : un administrateur authentifié configure une notification qui exécute un programme ou un script, et un paramètre de commande forgé s'exécute sous le compte de service PRTG, qui est SYSTEM, offrant une exécution de code à distance depuis une connexion web."
keywords:
  - prtg
  - prtgadmin
  - notification
  - command injection
  - windows monitoring
---

# PRTG Network Monitor

PRTG Network Monitor est une plateforme de supervision sous Windows largement utilisée, dotée d'une interface web (sur 80/443, et 8080/8443 pour le serveur central). Elle est attaquée principalement via son authentification web et sa fonctionnalité de notification. L'administrateur par défaut est `prtgadmin`/`prtgadmin`, fréquemment laissé inchangé. Une fois authentifié en tant qu'administrateur, les notifications de PRTG, qui peuvent exécuter un programme ou un script externe lorsqu'une alerte se déclenche, sont détournables : un paramètre de commande forgé dans une notification s'exécute sur le serveur, et comme le service central de PRTG s'exécute en tant que Local System, cette exécution a lieu en tant que SYSTEM. Ainsi, une connexion web (identifiants par défaut ou faibles) s'enchaîne directement vers une exécution de code à distance de niveau SYSTEM sur le serveur de supervision, qui détient par ailleurs les identifiants que PRTG utilise pour superviser le parc.

```bash
curl -sk https://<target>/index.htm | grep -i prtg          # fingerprint
curl -sk https://<target>:8080/public/checklogin.htm --data 'loginurl=&username=prtgadmin&password=prtgadmin'
```

## Sous-thèmes

- **[Énumération](enumeration.md)** : identification du produit et de la version.
- **[Authentification](authentication.md)** : le compte prtgadmin par défaut et les identifiants faibles.
- **[Abus des commandes de notification](notification-command-abuse.md)** : RCE SYSTEM via les notifications.
- **[Exploits connus](known-exploits.md)** : les vulnérabilités PRTG connues.

## Références

- [Documentation PRTG](https://www.paessler.com/manuals/prtg)
- [HackTricks : PRTG](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/prtg)
