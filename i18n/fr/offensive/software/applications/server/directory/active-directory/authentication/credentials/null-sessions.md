---
order: 20
title: "Sessions null : énumération anonyme et RID cycling"
description: "Énumérer un domaine sans aucun identifiant en abusant de l'accès SMB et LDAP anonyme : connexions IPC$ en session null, binds LDAP anonymes et RID cycling via LSARPC/SAMR pour reconstituer la liste des utilisateurs là où l'accès anonyme n'a jamais été verrouillé."
keywords:
  - session null
  - énumération anonyme
  - RID cycling
  - RestrictAnonymous
  - rpcclient
---

# Sessions null

Avant tout identifiant, il vaut la peine de se demander ce que le domaine divulgue à **personne du tout**. Une **session null** est une connexion non authentifiée au partage `IPC$` d'un hôte Windows (nom d'utilisateur et mot de passe vides) qui exposait historiquement les utilisateurs, groupes, partages et stratégies via SMB/RPC. Les binds **LDAP** anonymes et le **RID cycling** anonyme reposent sur la même idée appliquée à l'annuaire. Ce sont les plus anciennes techniques d'énumération de l'arsenal Windows, et elles restent payantes partout où les restrictions d'accès anonyme n'ont jamais été renforcées.

## Un verrouillage de longue date

Les sessions null étaient grandes ouvertes sur NT 4.0 et Windows 2000, où l'accès anonyme à `IPC$` renvoyait l'intégralité de la SAM. Microsoft les a progressivement restreintes avec `RestrictAnonymous`, `RestrictAnonymousSAM` et `everyoneincludesanonymous`, si bien que les serveurs membres modernes refusent généralement les appels intéressants. Mais les contrôleurs de domaine répondent encore à certaines requêtes LSARPC/SAMR anonymes par conception, les systèmes legacy et les appliances conservent souvent les sessions null activées, et les binds LDAP anonymes réapparaissent sur des DC mal configurés, de sorte que la technique n'a jamais totalement disparu.

## SMB anonyme et RID cycling

```bash
# Null-session connection and enumeration
rpcclient -U "" -N <target>
#  rpcclient $> enumdomusers ; querydispinfo ; lsaenumsid

# RID cycling: resolve SIDs to names to rebuild the user list without credentials
nxc smb <target> -u '' -p '' --rid-brute
enum4linux-ng -A <target>
```

## LDAP anonyme

```bash
# Anonymous bind against the DC; where allowed, dump naming contexts and accounts
ldapsearch -x -H ldap://<dc> -b "DC=example,DC=local" "(objectClass=user)" sAMAccountName
```

## Notes d'exploitation

- Le gain est une **liste d'utilisateurs et une stratégie de domaine sans aucun identifiant**, ce qui amorce le [password spraying](password-spraying.md) et l'[AS-REP roasting](../kerberos/roasting.md).
- Le **RID cycling** fonctionne même lorsque `enumdomusers` est bloqué, car résoudre à la main SID-500, SID-1000, SID-1001 ... renvoie toujours des noms là où SAMR/LSARPC répondent de façon anonyme.
- Les contrôleurs de domaine divulguent souvent plus d'informations de façon anonyme que les serveurs membres, testez donc toujours le DC directement.
- Une liste d'utilisateurs issue d'une [session null](user-and-group-enumeration.md) a souvent une valeur indiscernable de celle d'une liste authentifiée pour l'étape suivante, il s'agit donc d'un véritable point d'appui antérieur aux identifiants, et pas d'une simple curiosité.

## Outils

- **rpcclient** (`-U "" -N`) : énumération RPC brute en session null.
- **enum4linux-ng** : encapsule les vérifications classiques de session null et de RID cycling.
- **NetExec** (`-u '' -p '' --rid-brute`, `--users`, `--shares`) : énumération SMB anonyme.
- **ldapsearch** (`-x`) : binds LDAP anonymes.

## Références

- [enum4linux-ng (cddmp)](https://github.com/cddmp/enum4linux-ng)
- [HackTricks: SMB enumeration and null sessions](https://hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html)
- [Microsoft: Network access, do not allow anonymous enumeration of SAM accounts](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/network-access-do-not-allow-anonymous-enumeration-of-sam-accounts)
