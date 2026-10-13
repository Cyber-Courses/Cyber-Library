---
order: 18
title: "Énumération LDAP : interroger directement les objets Active Directory"
description: "Lire Active Directory via LDAP : requêtes authentifiées et anonymes, filtres de recherche utiles pour les utilisateurs, ordinateurs, groupes et comptes de service, et extractions massives de l'annuaire."
keywords:
  - énumération LDAP
  - ldapsearch
  - filtre LDAP
  - userAccountControl
  - ldapdomaindump
---

# Énumération LDAP

AD expose la quasi-totalité de son contenu via LDAP, et tout utilisateur de domaine authentifié peut en lire la grande majorité. Des requêtes LDAP ciblées sont le moyen d'extraire des tranches spécifiques (comptes de service, ordinateurs avec délégation, utilisateurs avec drapeaux faibles) sans collecter tout le graphe.

## Interroger avec des filtres

Les filtres de recherche LDAP sélectionnent les objets par attribut. Les attributs de prédilection sont `objectClass`, `sAMAccountName`, et le champ de bits `userAccountControl` (UAC) :

```bash
# All users
ldapsearch -x -H ldap://<dc> -D 'EXAMPLE\user' -w 'Password1' \
  -b 'DC=example,DC=local' '(&(objectClass=user)(objectCategory=person))' sAMAccountName

# All computers
ldapsearch ... '(objectClass=computer)' dNSHostName operatingSystem

# Domain admins (memberOf the group)
ldapsearch ... '(memberOf=CN=Domain Admins,CN=Users,DC=example,DC=local)' sAMAccountName
```

Les filtres sur les bits UAC font ressortir directement les cibles de plus grande valeur :

```
# Kerberoastable: user accounts with a servicePrincipalName (objectCategory=person
# excludes computer accounts, which are a subclass of user and do not crack)
(&(objectClass=user)(objectCategory=person)(servicePrincipalName=*)(!(sAMAccountName=krbtgt)))

# AS-REP roastable: DONT_REQ_PREAUTH (0x400000)
(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))

# Unconstrained delegation: TRUSTED_FOR_DELEGATION (0x80000)
(userAccountControl:1.2.840.113556.1.4.803:=524288)

# Password never expires (0x10000) OR password not required (0x20): one bit test per flag
(|(userAccountControl:1.2.840.113556.1.4.803:=65536)(userAccountControl:1.2.840.113556.1.4.803:=32))

# MachineAccountQuota: computers any user may create; read ms-DS-MachineAccountQuota on the
# domain root (a non-zero value gates RBCD, noPac, shadow-credentials-on-new-computer, Certifried)
```

L'OID `1.2.840.113556.1.4.803` est la règle de correspondance ET-bit-à-bit, essentielle pour lire les drapeaux UAC.

## Extractions massives

Pour une copie hors ligne complète, extrayez chaque objet et attribut :

```bash
# ldapdomaindump: HTML/JSON/greppable dump of users, groups, computers, policies
ldapdomaindump -u 'EXAMPLE\user' -p 'Password1' <dc-ip>

# NetExec LDAP module: targeted extractions
nxc ldap <dc> -u user -p pass --users --groups --computers
nxc ldap <dc> -u user -p pass --kerberoasting out.txt --asreproast out.txt
```

## Lectures anonymes et sessions nulles

Certains DC autorisent les binds LDAP anonymes ou exposent des données via des sessions SMB nulles. Lorsque c'est permis, le rootDSE, le schéma et parfois les données d'objets se lisent sans identifiants (voir [bind LDAP anonyme](../../../ldap/protocol/anonymous-bind-and-enumeration.md)). Même lorsque la lecture des objets nécessite une authentification, la découverte des contextes de nommage via le rootDSE n'en a pas besoin.

## Notes d'exploitation

- Lisez les attributs intéressants, pas seulement les noms : les champs `description` et `info` contiennent fréquemment des mots de passe ; `ms-MCS-AdmPwd` (LAPS) et `msDS-ManagedPassword` (gMSA) détiennent les identifiants machine et de service pour les principaux autorisés à les lire (voir [identifiants dans les attributs](../../../ldap/protocol/credentials-in-attributes.md)).
- Interrogez le catalogue global (port 3268) pour une vue à l'échelle de la forêt couvrant tous les domaines en une seule recherche.
- Tout ce que vous extrayez ici alimente [BloodHound](../../dacl/bloodhound.md) ; les filtres ciblés ci-dessus servent quand vous voulez une réponse précise rapidement.

## Outils

- **ldapsearch** : requêtes filtrées brutes.
- **ldapdomaindump** : extraction hors ligne complète de l'annuaire.
- **NetExec (nxc) ldap** : utilisateurs/groupes/ordinateurs ciblés et extraction de roasting.
- **windapsearch** : requêtes pratiques (utilisateurs privilégiés, ordinateurs, GPO).

## Références

- [NetExec: LDAP enumeration](https://github.com/Pennyw0rth/NetExec)
- [ldapdomaindump (dirkjanm)](https://github.com/dirkjanm/ldapdomaindump)
- [windapsearch (ropnop)](https://github.com/ropnop/windapsearch)
- [Microsoft: userAccountControl attribute flags](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/useraccountcontrol-manipulate-account-properties)
