---
title: "Abus des commandes de notification : RCE SYSTEM via les notifications PRTG"
order: 2
description: "Les notifications PRTG peuvent exécuter un programme ou un script externe lorsqu'elles sont déclenchées, et le paramètre de commande est contrôlé par un administrateur authentifié. Un paramètre forgé injecte des commandes système qui s'exécutent sous le compte de service central de PRTG, Local System, de sorte qu'un administrateur crée ou modifie une notification de type Execute Program et la déclenche pour obtenir l'exécution de code en tant que SYSTEM sur le serveur."
keywords:
  - prtg notification
  - execute program
  - command injection
  - system
  - rce
---

# Abus des commandes de notification

Le système de notification de PRTG peut exécuter un programme ou un script externe comme action (le type de notification « Execute Program » / démo), et les paramètres qui lui sont transmis sont contrôlés par un administrateur authentifié via l'interface web. Parce que ces paramètres sont placés dans une commande exécutée par la sonde/le serveur central de PRTG, un paramètre forgé contenant de la syntaxe de commande système injecte des commandes supplémentaires, et comme le service central de PRTG s'exécute en tant que Local System sous Windows, les commandes injectées s'exécutent en tant que SYSTEM. L'attaque est donc la suivante : en tant qu'administrateur, créer ou modifier une notification de type execute-program avec un paramètre portant une charge utile, puis déclencher la notification (la tester manuellement, ou faire entrer un capteur dans un état qui la déclenche), et la charge utile s'exécute en tant que SYSTEM sur le serveur de supervision.

```bash
# authenticated (prtgadmin). Create/edit an "Execute Program" notification whose
# parameter field carries a command payload, then trigger it. The parameter is run
# by the core (as SYSTEM); injecting PowerShell/command syntax executes it, e.g.:
#   parameter:  test.txt;net user pwn P@ss1 /add;net localgroup administrators pwn /add
# the historic auth command-injection (CVE class) abused exactly this notification
# parameter to run commands as SYSTEM. Tools/metasploit automate create+trigger.
curl -sk -b cj 'https://<target>:8080/editnotification.htm' --data '<notification-with-payload>'
curl -sk -b cj 'https://<target>:8080/api/notificationtest.htm?id=<notifid>'   # trigger
```

## Notes d'exploitation

- La condition préalable est simplement une session administrateur : le paramètre de notification est contrôlé par l'administrateur et aboutit dans une commande exécutée en tant que SYSTEM, de sorte qu'administrateur équivaut à une RCE SYSTEM.
- Déclenchez la notification pour exécuter la charge utile : PRTG propose une action de test/exécution pour les notifications, ou un changement d'état de capteur l'invoque ; l'action de test est la plus simple.
- L'exécution a lieu en tant que SYSTEM sur le serveur de supervision Windows, qui stocke par ailleurs les identifiants que PRTG utilise pour superviser les équipements (et est souvent joint au domaine), ce qui en fait un fort point de pivot ; ajoutez un administrateur local ou exécutez directement une charge utile.
- Il s'agit de la RCE canonique de PRTG ; l'injection précise et tout encodage requis dépendent de la version, voir [Exploits connus](known-exploits.md).

## Références

- [PRTG : notifications](https://www.paessler.com/manuals/prtg/notifications_settings)
- [HackTricks : RCE PRTG](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/prtg)
