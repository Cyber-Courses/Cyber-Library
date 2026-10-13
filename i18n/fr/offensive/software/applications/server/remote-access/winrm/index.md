---
order: 8
title: "WinRM : attaquer Windows Remote Management"
description: "WinRM implémente WS-Management sur les ports TCP 5985 (HTTP) et 5986 (HTTPS), fournissant l'exécution de commandes à distance sous Windows via PowerShell Remoting. La surface est l'énumération de service, l'authentification par mots de passe ou par hashes NTLM (pass-the-hash) et l'exécution de commandes : des identifiants valides ou un hash pour un utilisateur Remote Management donnent un shell interactif, typiquement avec des droits administratifs."
keywords:
  - winrm
  - ws-management
  - powershell remoting
  - evil-winrm
  - port 5985
---

# WinRM

WinRM (Windows Remote Management) est l'implémentation par Microsoft du protocole WS-Management, à l'écoute sur les ports TCP 5985 (HTTP) et 5986 (HTTPS), et c'est le transport derrière PowerShell Remoting. C'est une cible privilégiée de mouvement latéral et d'exécution à distance sur les réseaux Windows : avec des identifiants valides (ou un hash NT, puisque WinRM prend en charge l'authentification NTLM et donc le pass-the-hash) pour un membre du groupe Remote Management Users ou un administrateur local, un attaquant obtient une session PowerShell interactive sur l'hôte, généralement avec des privilèges administratifs. La surface est l'énumération du service, l'obtention et l'utilisation d'identifiants ou de hashes, et l'exécution de commandes elle-même.

```bash
nmap -p5985,5986 -sV <target>
nxc winrm <target> -u user -p 'pass'           # validates creds and flags Pwn3d! (admin)
```

## Sous-thèmes

- **[Énumération](enumeration.md)** : détecter WinRM et sa configuration.
- **[Authentification](authentication.md)** : mots de passe, NTLM et pass-the-hash.
- **[Exécution de commandes](command-execution.md)** : obtenir un shell avec Evil-WinRM et PowerShell Remoting.

## Références

- [MS-WSMV (WS-Management)](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-wsmv/)
- [HackTricks: WinRM (5985)](https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm)
