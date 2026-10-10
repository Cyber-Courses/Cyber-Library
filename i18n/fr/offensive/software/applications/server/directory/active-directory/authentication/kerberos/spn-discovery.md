---
title: "Découverte de SPN : localiser les comptes de service pour le roasting"
order: 9
description: "Trouver les service principal names dans Active Directory pour identifier les comptes kerberoastables : interrogation de servicePrincipalName via LDAP, setspn, et le recoupement avec les identités de service à forte valeur."
keywords:
  - SPN
  - servicePrincipalName
  - kerberoasting target
  - setspn
  - service account
---

# Découverte de SPN

Un **service principal name** (SPN) associe une instance de service au compte qui l'exécute. Pour tout compte doté d'un SPN, n'importe quel utilisateur du domaine peut demander un ticket de service, et ce ticket est chiffré avec le hash du mot de passe du compte, ce qui constitue le prérequis du Kerberoasting. Découvrir quels comptes portent des SPN vous indique exactement quelles identités sont roastables, et les plus précieuses sont généralement des comptes de service à privilèges élevés et aux mots de passe anciens, choisis par un humain.

## Trouver les comptes dotés de SPN

La requête se résume à un seul filtre LDAP : les objets utilisateur dont le `servicePrincipalName` est défini.

```bash
# LDAP filter for kerberoastable users. objectCategory=person excludes computer
# accounts (computer is a subclass of user, and machine-account hashes do not crack)
ldapsearch ... '(&(objectClass=user)(objectCategory=person)(servicePrincipalName=*)(!(sAMAccountName=krbtgt)))' \
  sAMAccountName servicePrincipalName memberOf

# Tooling that lists SPNs and their accounts
nxc ldap <dc> -u user -p pass --kerberoasting out.txt
GetUserSPNs.py example.local/user:pass -dc-ip <dc>      # add -request to also roast
setspn -T example.local -Q */*                          # from a domain-joined host
```

Depuis un hôte du domaine, PowerView :

```
Get-DomainUser -SPN -Properties samaccountname,serviceprincipalname,memberof,pwdlastset
```

## Lire les résultats

Priorisez selon deux signaux :

- **Privilège** : recoupez le `memberOf` du compte avec les groupes privilégiés. Un compte de service membre de Domain Admins (courant pour les anciens services SQL, de sauvegarde ou de supervision) rend un roast immédiatement critique pour le domaine.
- **Ancienneté du mot de passe** : un `pwdLastSet` ancien sur un compte de service géré par un humain suggère un mot de passe statique et cassable. Les comptes machine et les gMSA ont des mots de passe aléatoires longs et ne valent pas la peine d'être roastés.

Notez la différence entre les SPN d'utilisateur (roastables) et les SPN des comptes machine : les hashs de comptes machine sont de fait incassables, le Kerberoasting cible donc les comptes *utilisateur* dotés de SPN.

## Notes d'exploitation

- Trouver le SPN relève de l'énumération ; demander et casser le ticket est traité sous la rubrique roasting Kerberos dans la section authentification et identifiants.
- Là où un principal que vous contrôlez dispose de `GenericAll`/`GenericWrite` sur un utilisateur cible, vous pouvez *écrire* un SPN dessus et le roaster à la demande (Kerberoasting ciblé), de sorte que la découverte de SPN éclaire aussi le choix des primitives d'écriture à exploiter.
- Un seul utilisateur de domaine à faibles privilèges suffit pour énumérer chaque SPN du domaine.

## Outils

- **GetUserSPNs.py** (Impacket) : lister et demander des tickets de service.
- **NetExec (nxc) ldap --kerberoasting** : énumérer et extraire.
- **PowerView `Get-DomainUser -SPN`** : découverte depuis l'hôte.
- **setspn** : requête SPN native Windows.

## Références

- [GhostPack Rubeus (kerberoast)](https://github.com/GhostPack/Rubeus)
- [Impacket GetUserSPNs](https://github.com/fortra/impacket)
- [Microsoft: service principal names](https://learn.microsoft.com/en-us/windows/win32/ad/service-principal-names)
