---
title: "Authentification : le compte prtgadmin par défaut et les identifiants faibles"
order: 1
description: "PRTG est livré avec un compte administrateur par défaut, prtgadmin/prtgadmin, fréquemment laissé inchangé, et accepte par ailleurs des mots de passe faibles. Une connexion web en tant qu'administrateur est l'unique condition préalable à l'exécution de code en tant que SYSTEM basée sur les notifications, de sorte que les identifiants par défaut ou pulvérisés contre la page de connexion de PRTG constituent la voie directe pour compromettre le serveur."
keywords:
  - prtgadmin
  - default credentials
  - weak password
  - prtg login
  - administrator
---

# Authentification

L'interface web de PRTG s'authentifie avec un nom d'utilisateur et un mot de passe, et la voie d'entrée passe presque toujours par les identifiants. L'administrateur par défaut est `prtgadmin` avec le mot de passe `prtgadmin`, laissé inchangé sur une grande proportion des installations. Lorsqu'il est modifié, les mots de passe faibles ou réutilisés sont courants, et les anciennes versions de PRTG n'appliquent aucune politique stricte. Parce que l'accès administrateur est l'unique condition préalable à l'exécution de code basée sur les notifications qui s'exécute en tant que SYSTEM, une connexion réussie équivaut de fait à une compromission du serveur, de sorte que l'essai du mot de passe par défaut et la pulvérisation de quelques mots de passe faibles contre la page de connexion constituent l'attaque principale.

```bash
# default admin login (core on 8080 in this example)
curl -sk -c cj 'https://<target>:8080/public/checklogin.htm' \
  --data 'loginurl=/&username=prtgadmin&password=prtgadmin'
# a successful login sets session cookies used for the authenticated API/UI
curl -sk -b cj 'https://<target>:8080/api/table.json?content=sensors&columns=objid,device'
```

## Notes d'exploitation

- Essayez `prtgadmin`/`prtgadmin` en premier ; c'est la valeur par défaut canonique et elle accorde le statut administrateur, soit tout ce qui est nécessaire pour l'[abus des commandes de notification](notification-command-abuse.md).
- Les mots de passe faibles/réutilisés et l'absence de verrouillage sur les anciennes versions rendent la pulvérisation efficace ; un seul identifiant administrateur équivaut à une RCE de niveau SYSTEM, de sorte que la page de connexion est de grande valeur.
- La connexion définit des cookies de session utilisés pour l'API authentifiée (`api/table.json`, gestion des notifications), qui pilotent la suite.
- Une session administrateur mène directement à l'[abus des commandes de notification](notification-command-abuse.md) ; un accès non administrateur est de peu d'utilité ici.

## Références

- [PRTG : comptes utilisateurs](https://www.paessler.com/manuals/prtg/user_accounts_settings)
- [HackTricks : PRTG](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/prtg)
