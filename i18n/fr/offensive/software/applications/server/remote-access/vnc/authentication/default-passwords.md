---
title: "Mots de passe par défaut : mots de passe VNC constructeur et devinables"
description: "VNC sur les appliances et les outils préinstallés utilise fréquemment un mot de passe fixe ou devinable, et le mot de passe VNC classique est plafonné à huit caractères, réduisant davantage l'espace. Essayer les valeurs par défaut constructeur et les mots de passe VNC courants donne souvent un accès au bureau, et les fichiers de mots de passe VNC stockés peuvent être déchiffrés avec la clé DES fixe et publiée."
keywords:
  - mot de passe par défaut vnc
  - limite de huit caractères
  - mot de passe stocké
  - vncpasswd
  - clé fixe
---

# Mots de passe par défaut

Là où un serveur VNC utilise bien le schéma par mot de passe, ce mot de passe est souvent faible : les appliances, les équipements KVM-over-IP et les outils distants embarqués sont livrés avec des valeurs par défaut fixes ou documentées, et le mot de passe VNC classique est tronqué à huit caractères, réduisant l'espace de clés. Essayer le mot de passe par défaut de l'appareil et les mots de passe VNC courants fonctionne fréquemment. Séparément, VNC stocke son mot de passe sur disque obfusqué avec une clé DES fixe et publiquement connue (et non hachée), de sorte qu'un fichier de mots de passe récupéré est déchiffré directement vers le mot de passe en clair, qui est ensuite réutilisé.

```bash
# try defaults/common passwords (8-char cap means short lists are effective)
vncviewer <target>::5900        # enter documented default or common password
nmap -p5900 --script vnc-brute --script-args passdb=vnc-defaults.txt <target>
# decrypt a stored VNC password file (fixed DES key, reversible)
#   files: ~/.vnc/passwd, UltraVNC.ini, registry values
echo -n "<hex-of-passwd-file>" | xxd -r -p | openssl enc -d -des-cbc -K e84ad660c4721ae0 -iv 0000000000000000 2>/dev/null
vncpwd passwd      # tools that apply the known key directly
```

## Notes d'exploitation

- La troncature à huit caractères signifie que seuls les huit premiers octets de tout mot de passe comptent, de sorte que les courtes listes de valeurs par défaut/courantes sont étonnamment efficaces ; identifiez l'appareil pour sa valeur par défaut documentée.
- Les mots de passe VNC stockés sont obfusqués, non hachés : la clé DES est fixe et publique, de sorte que tout fichier `passwd`, `UltraVNC.ini` ou valeur de registre récupéré se déchiffre directement en clair avec un outil à clé connue.
- Un mot de passe VNC déchiffré est réutilisable, testez-le sur d'autres points de terminaison VNC et envisagez-le pour d'autres services, puisque les opérateurs le réutilisent.
- Là où la force brute est nécessaire plutôt que les valeurs par défaut, voir [Force brute de mot de passe](password-brute-force.md) ; là où le serveur n'a aucun mot de passe, voir [Aucune authentification](no-authentication.md).

## Outils

- [vncpwd (fixed-key VNC password decryptor)](https://github.com/jeroennijhof/vncpwd)

## Références

- [VNC password storage and fixed key](https://www.vidarholen.net/contents/junk/vnc.html)
- [HackTricks: VNC](https://book.hacktricks.xyz/network-services-pentesting/pentesting-vnc)
