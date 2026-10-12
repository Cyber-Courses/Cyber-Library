---
title: "Redirection du presse-papiers : capturer des données via le canal presse-papiers RDP"
description: "Le canal virtuel de redirection du presse-papiers de RDP synchronise le presse-papiers entre le client et le serveur par commodité. Un attaquant qui contrôle l'une des extrémités surveille et capture le presse-papiers synchronisé, qui transporte couramment des mots de passe copiés depuis des gestionnaires, des jetons et du texte sensible, et peut injecter du contenu, transformant un presse-papiers partagé en canal de vol de données et d'influence."
keywords:
  - clipboard redirection
  - cliprdr
  - rdp channel
  - data theft
  - clipboard monitoring
---

# Redirection du presse-papiers

RDP synchronise le presse-papiers entre le client et la session distante via le canal virtuel `cliprdr`, de sorte qu'une copie d'un côté est disponible de l'autre. Cette commodité est un canal de données qu'un attaquant détourne depuis l'extrémité qu'il contrôle. Surveiller le presse-papiers synchronisé capture ce que les utilisateurs copient, ce qui inclut très souvent des mots de passe collés depuis un gestionnaire de mots de passe, des jetons d'API, des codes à usage unique et du texte sensible, et le canal est bidirectionnel, de sorte qu'un attaquant peut aussi injecter du contenu dans le presse-papiers pour influencer ce qu'un utilisateur colle. Sur un serveur de terminaux compromis, ou un point d'accès RDP malveillant auquel un utilisateur se connecte, le presse-papiers devient une surface de vol et de manipulation.

```bash
# on a controlled RDP host/session, poll the clipboard for secrets
# (PowerShell in the session)
while ($true) { Get-Clipboard; Start-Sleep -Seconds 2 }     # capture what users copy
# a malicious RDP server/endpoint can likewise log the cliprdr channel contents
# injection: set the clipboard to attacker content the user may paste
Set-Clipboard -Value 'attacker-controlled'
```

## Notes d'exploitation

- Les gestionnaires de mots de passe encouragent le copier-coller des identifiants, donc un presse-papiers RDP surveillé est une source d'identifiants fiable ; interrogez-le sur un hôte où les utilisateurs ont des sessions, ou sur un point d'accès malveillant auquel ils se connectent.
- L'injection est l'abus le plus subtil : remplacer le contenu du presse-papiers (par exemple échanger une commande collée ou une adresse de cryptomonnaie) manipule ce sur quoi l'utilisateur agit.
- Cela nécessite le contrôle d'une extrémité de la session ; sur un serveur de terminaux compromis, vous lisez le presse-papiers de chaque utilisateur, et une destination RDP malveillante lit celui du client qui se connecte.
- La redirection du presse-papiers est activée par défaut dans de nombreuses configurations ; sa présence fait partie de la surface de redirection de périphériques de la session, aux côtés de la [redirection de lecteurs](drive-redirection.md).

## Références

- [MS-RDPECLIP (clipboard virtual channel)](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rdpeclip/)
- [HackTricks: RDP redirection](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
