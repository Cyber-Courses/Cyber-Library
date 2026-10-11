---
title: "SMB : attaquer le protocole de partage de fichiers de Windows"
description: "SMB (Server Message Block) sert des partages de fichiers Windows sur TCP 445 et soutient une grande partie d'un réseau Windows. La surface offensive est l'établissement de session (sessions null et invité), l'énumération des partages et des permissions, le butinage des partages lisibles et l'empoisonnement de ceux inscriptibles, les faiblesses de signature des messages et de relais NTLM, et les failles d'exécution de code à distance du protocole lui-même."
keywords:
  - smb
  - port 445
  - relais ntlm
  - énumération des partages
  - session null
---

# SMB

SMB est le protocole Server Message Block, servant les partages de fichiers Windows (et Samba) sur TCP 445, et il est central dans les environnements Windows : les partages de fichiers, les partages de domaine `SYSVOL`/`NETLOGON`, les tubes nommés pour RPC et les partages administratifs (`C$`, `ADMIN$`) reposent tous dessus. D'un point de vue offensif, c'est l'un des services les plus riches sur un réseau. La surface couvre la façon dont une session est établie (sessions null anonymes, accès invité ou identifiants), l'énumération des partages et de l'accès par identité à ceux-ci, le butinage des partages lisibles et l'empoisonnement de ceux inscriptibles pour capturer des identifiants ou planter du code, les faiblesses de signature des messages et de relais NTLM qui transforment l'authentification en déplacement latéral, et les bugs d'exécution de code à distance de l'implémentation du protocole elle-même.

```bash
# fingerprint and reach the service
nmap -p445 --script smb-protocols,smb2-security-mode <target>
nxc smb <target>                              # OS, domain, signing, SMBv1 status
```

## Sous-sujets

- **[Énumération des partages](share-enumeration.md)** : lister les partages et l'accès par identité.
- **[Session null](null-session.md)** : accès anonyme aux partages et à RPC.
- **[Signature et relais](signing-and-relay.md)** : faiblesses de la signature SMB et relais NTLM.
- **[Butin et fichiers sensibles](loot-and-sensitive-files/index.md)** : exploiter les partages lisibles.
- **[Empoisonnement des partages inscriptibles](writable-share-poisoning/index.md)** : planter des charges utiles et des fichiers de coercition.
- **[Exploits SMB connus](known-smb-exploits.md)** : les failles d'exécution de code à distance du protocole.

## Références

- [MS-SMB2 protocol specification](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/)
- [NetExec: SMB protocol](https://www.netexec.wiki/smb-protocol)
- [Impacket](https://github.com/fortra/impacket)
