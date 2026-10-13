---
title: "rexec : attaquer le service d'exécution distante"
order: 3
description: "rexec (rexecd sur TCP 512) exécute une commande sur un hôte distant, en s'authentifiant avec un nom d'utilisateur et un mot de passe envoyés en clair, et peut aussi honorer la confiance d'hôte. Les attaques sont la capture de l'identifiant en clair, l'abus des relations d'hôtes de confiance pour une exécution sans mot de passe, et l'injection de commande là où la commande exécutée est construite de manière non sûre."
keywords:
  - rexec
  - rexecd
  - port 512
  - cleartext
  - remote execution
---

# rexec

`rexec` exécute une commande sur un hôte distant via `rexecd` sur TCP 512. Contrairement à rsh, il a été conçu autour d'un nom d'utilisateur/mot de passe, envoyé en clair, et certaines implémentations honorent aussi les fichiers de confiance basés sur l'hôte. Les attaques découlent de ces deux faits. L'identifiant en clair est capturé par un attaquant bien positionné. Les relations d'hôtes de confiance, lorsqu'elles sont honorées, donnent une exécution de commande sans mot de passe tout comme avec rsh. Et là où le serveur ou un wrapper construit la commande exécutée à partir d'une entrée influencée par l'attaquant de manière non sûre, l'injection de commande étend ce qui s'exécute. Un rexec réussi exécute une commande en tant qu'utilisateur authentifié.

```bash
# run a command with credentials (sent in cleartext)
rexec -l <user> -p <password> <target> id
nmap -p512 -sV <target>
```

## Sous-thèmes

- **[Mots de passe en clair](cleartext-passwords.md)** : capturer l'identifiant rexec.
- **[Contournement d'hôte de confiance](trusted-host-bypass.md)** : exécution sans mot de passe via la confiance d'hôte.
- **[Injection de commande](command-injection.md)** : abuser d'une construction de commande non sûre.

## Références

- [rexecd manual](https://linux.die.net/man/8/rexecd)
- [HackTricks: rexec (512)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
