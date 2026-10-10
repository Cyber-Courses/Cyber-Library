---
title: "Session nulle : accès anonyme aux partages SMB et à RPC"
order: 1
description: "Une session nulle est une connexion SMB non authentifiée (nom d'utilisateur et mot de passe vides) qui, lorsqu'elle est autorisée, donne accès au tube IPC$ et parfois aux partages. Via IPC$, elle atteint les interfaces RPC qui énumèrent les utilisateurs, les groupes, les partages et la politique de mots de passe, faisant d'une session nulle autorisée un riche point d'appui de reconnaissance avant l'obtention d'identifiants."
keywords:
  - null session
  - ipc$
  - rpcclient
  - anonymous
  - enumeration
---

# Session nulle

Une session nulle est une connexion SMB établie avec un nom d'utilisateur et un mot de passe vides. Historiquement, Windows accordait aux sessions nulles un accès étendu ; les systèmes modernes les restreignent, mais elles restent fréquemment autorisées sur le partage `IPC$` (le point de terminaison des tubes nommés), et les serveurs mal configurés ainsi que de nombreux déploiements Samba en permettent davantage. L'intérêt réside dans ce qu'expose `IPC$` : les interfaces MS-RPC accessibles via les tubes nommés, à travers lesquelles un client non authentifié énumère les utilisateurs, les groupes, les partages et la politique de comptes, une reconnaissance qui nécessite normalement des identifiants.

```bash
# test and use a null session
nxc smb <target> -u '' -p '' --shares           # anonymous share access
rpcclient -U '' -N <target>                      # null RPC session
#   rpcclient $> srvinfo            # server type/version
#   rpcclient $> enumdomusers       # domain/local users (SID + name)
#   rpcclient $> enumdomgroups      # groups
#   rpcclient $> querydominfo       # domain info
#   rpcclient $> getdompwinfo       # password policy (lockout, min length)
#   rpcclient $> lsaquery           # domain SID
```

## Cyclage des RID

Même lorsque `enumdomusers` est bloqué, une session nulle (ou invitée) capable de résoudre des SID énumère les utilisateurs en faisant défiler les identifiants relatifs face au SID du domaine :

```bash
# brute-force RIDs to recover usernames when direct enumeration is denied
nxc smb <target> -u '' -p '' --rid-brute
# or via rpcclient: lookupsids S-1-5-21-...-500, -501, -1000, -1001, ...
lookupnames <sid>   # map names to SIDs and back within rpcclient
```

Le cyclage des RID fonctionne parce que les SID de comptes sont le SID du domaine suivi d'un RID séquentiel (500 = Administrator, 501 = Guest, 1000+ = comptes créés), de sorte que résoudre chaque RID permet de récupérer le nom même sans droit de listage.

## Notes d'exploitation

- La liste d'utilisateurs récupérée alimente le password spraying et le ciblage AS-REP/Kerberoast ; la politique de mots de passe (issue de `getdompwinfo`) fixe un seuil de spraying sûr pour éviter les verrouillages.
- L'accès par session nulle à `IPC$` est courant même là où les partages de données sont verrouillés, donc testez-le toujours ; les serveurs Samba en permettent souvent plus que Windows.
- Le cyclage des RID est le repli lorsque `enumdomusers` est restreint mais que les résolutions de SID sont autorisées ; `--rid-brute` l'automatise.
- Une session nulle relève de la reconnaissance, pas de l'accès aux données ; combinez la liste d'utilisateurs et la politique avec l'énumération des partages et les chemins d'attaque du domaine.

## Outils

- [rpcclient (Samba)](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html)
- [NetExec](https://github.com/Pennyw0rth/NetExec)
- [enum4linux-ng](https://github.com/cddmp/enum4linux-ng)

## Références

- [NetExec : énumération SMB](https://www.netexec.wiki/smb-protocol/enumeration)
- [MS-SRVS et MS-SAMR (RPC via IPC$)](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-samr/)
