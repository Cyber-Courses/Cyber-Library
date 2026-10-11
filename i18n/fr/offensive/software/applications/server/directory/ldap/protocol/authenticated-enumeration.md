---
title: "Énumération LDAP authentifiée : extraire l'ensemble complet des objets"
description: "Énumération d'un annuaire LDAP avec n'importe quel bind valide : vidage des utilisateurs, groupes et structure, pagination des grands ensembles de résultats, et interrogation du catalogue global pour une vue à l'échelle de la forêt."
keywords:
  - énumération authentifiée
  - bind LDAP
  - résultats paginés
  - catalogue global
  - ldapsearch
---

# Énumération authentifiée

N'importe quel identifiant valide, aussi peu privilégié soit-il, débloque généralement une lecture presque complète d'un annuaire. Sur AD, un seul utilisateur du domaine peut lire presque tous les objets ; sur OpenLDAP, un utilisateur authentifié lit ce que les ACL accordent, ce qui est souvent large. L'énumération authentifiée est l'étape où l'inventaire complet est constitué.

## Se lier et vider

```bash
# Simple bind (cleartext over LDAP; prefer LDAPS where available)
ldapsearch -x -H ldap://<target> -D 'cn=user,dc=example,dc=com' -w 'password' \
  -b 'dc=example,dc=com' "(objectClass=*)"

# AD-style bind
ldapsearch -x -H ldap://<dc> -D 'EXAMPLE\user' -w 'password' \
  -b 'DC=example,DC=local' "(objectClass=user)" sAMAccountName
```

Pour AD, les requêtes conscientes du modèle d'objets (filtres de bits UAC, SPN, délégation) sont traitées sous [énumération LDAP](../../active-directory/authentication/credentials/ldap-enumeration.md) ; les mécanismes ci-dessous s'appliquent à n'importe quel annuaire.

## Paginer les grands résultats

Les annuaires plafonnent le nombre d'entrées qu'une seule recherche renvoie (le `MaxPageSize` d'AD est 1000 par défaut), donc un vidage naïf tronque silencieusement. Utilisez le contrôle de résultats paginés simple :

```bash
# ldapsearch handles paging with -E pr=
ldapsearch ... -E pr=1000/noprompt "(objectClass=user)" sAMAccountName

# ldapdomaindump and nxc page automatically
ldapdomaindump -u 'EXAMPLE\user' -p 'password' <dc>
```

Si un vidage renvoie exactement 1000 (ou un autre nombre rond) d'entrées, supposez une troncature et passez à une requête paginée.

## Catalogue global

Le **catalogue global** d'AD (ports 3268 en clair, 3269 TLS) détient une réplique partielle à l'échelle de la forêt, de sorte qu'une seule requête contre lui énumère chaque domaine de la forêt d'un coup :

```bash
ldapsearch -x -H ldap://<gc>:3268 -D 'EXAMPLE\user' -w 'password' \
  -b '' -s sub "(objectClass=user)" sAMAccountName
```

## Notes d'exploitation

- Liez-vous via **LDAPS** (636) ou utilisez StartTLS quand c'est possible ; un bind simple sur du LDAP en clair envoie le mot de passe en clair et est lui-même capturable sur le réseau.
- Paginez toujours ; une énumération tronquée masque discrètement les comptes à forte valeur précis que vous recherchez.
- Demandez les attributs qui comptent (`description`, `memberOf`, UAC, `servicePrincipalName`) plutôt que des objets entiers, à la fois pour la rapidité et pour faire ressortir directement les [champs porteurs d'identifiants](credentials-in-attributes.md).

## Outils

- **ldapsearch** avec `-E pr=` : lectures authentifiées paginées.
- **ldapdomaindump** : vidage complet de l'annuaire hors ligne avec pagination.
- **NetExec (nxc) ldap** : extractions authentifiées ciblées.

## Références

- The Hacker Recipes: LDAP enumeration
- Microsoft: LDAP paging and MaxPageSize
