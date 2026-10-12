---
title: "Contournement d'hôte de confiance : rexec sans mot de passe via la confiance d'hôte"
order: 3
description: "Là où rexecd honore les fichiers de confiance basés sur l'hôte, ou là où le chemin rsh apparenté est atteignable, un attaquant exécute des commandes sans mot de passe en se connectant depuis, ou en usurpant, un hôte de confiance, ou en implantant une entrée .rhosts. Cela supprime entièrement l'exigence d'identifiant, donnant une exécution de commande en tant qu'utilisateur cible."
keywords:
  - trusted host
  - rexec
  - rhosts
  - passwordless
  - command execution
---

# Contournement d'hôte de confiance

Bien que rexec repose sur un mot de passe par conception, le modèle de confiance de Berkeley se superpose fréquemment : les systèmes exécutant rexec exécutent généralement aussi rsh, honorant `~/.rhosts` et `/etc/hosts.equiv`, et un attaquant qui établit la confiance exécute des commandes sans mot de passe via ce chemin de confiance. Les techniques sont celles qui sont partagées, se connecter depuis ou usurper un hôte de confiance, ou implanter une entrée `.rhosts` là où le répertoire personnel de la cible est accessible en écriture, après quoi l'exécution de commande ne nécessite aucun identifiant. Cela contourne entièrement le mot de passe en clair et constitue la voie la plus forte lorsque la confiance peut être obtenue.

```bash
# if trust applies (or via the co-located rsh path), execute with no password
rsh -l <user> <target> id                      # trusted-host execution, no credential
# plant trust first where the home is writable
echo '+ +' >> ~victim/.rhosts
# then run commands as victim with no password
```

## Notes d'exploitation

- Les hôtes rexec exposent presque toujours aussi rsh/rlogin avec les mêmes fichiers de confiance, de sorte que l'abus de confiance établi pour l'un s'applique à tous ; les techniques [rhosts](../rsh/rhosts-bypass.md) et [hosts.equiv](../trust-abuse/hosts-equiv-trust.md) se transposent directement.
- La confiance supprime l'exigence de mot de passe que rexec impose autrement, donc obtenir la confiance est préférable à capturer ou deviner l'identifiant.
- Implanter `.rhosts` nécessite une écriture dans le répertoire personnel ; un `hosts.equiv` permissif ou un hôte de confiance usurpable ne nécessite aucune écriture.
- Là où la confiance ne peut être obtenue, repliez-vous sur la [capture du mot de passe en clair](cleartext-passwords.md).

## Références

- [man 5 rhosts](https://man7.org/linux/man-pages/man5/rhosts.5.html)
- [HackTricks: rexec](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rexec)
