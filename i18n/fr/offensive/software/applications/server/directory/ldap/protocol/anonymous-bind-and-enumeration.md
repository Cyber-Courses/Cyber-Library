---
title: "Liaison anonyme et énumération : lire un annuaire sans identifiants"
description: "Énumérer les annuaires LDAP via les liaisons anonymes et nulles : lire le rootDSE, les contextes de nommage et les données d'objets sans authentification sur AD, OpenLDAP et d'autres annuaires."
keywords:
  - anonymous bind
  - null bind
  - rootDSE
  - LDAP enumeration
  - unauthenticated
---

# Liaison anonyme et énumération

Une **liaison anonyme** LDAP s'authentifie sans aucun identifiant, et de nombreux annuaires l'autorisent pour au moins une partie de l'arbre. Là où elle est permise, un attaquant non authentifié lit la structure de l'annuaire et souvent les données d'objets, ce qui constitue un solide point d'ancrage non authentifié.

## Tester l'accès anonyme

Établissez une liaison sans identifiant et essayez de lire la base :

```bash
# Anonymous bind, read the rootDSE (works on almost all directories)
ldapsearch -x -H ldap://<target> -s base -b "" "(objectClass=*)"

# Anonymous read of actual object data (works where the directory allows it)
ldapsearch -x -H ldap://<target> -b "DC=example,DC=local" "(objectClass=*)"
ldapsearch -x -H ldap://<target> -b "dc=example,dc=com" "(objectClass=person)" cn mail
```

Trois issues :

- La liaison réussit **et** renvoie des données d'objets : annuaire entièrement lisible anonymement.
- La liaison réussit mais les lectures d'objets ne renvoient rien ou une « operations error » : liaison anonyme autorisée, les lectures de données requièrent une authentification (courant sur les AD modernes, où le rootDSE reste lisible).
- La liaison est rejetée : liaisons anonymes désactivées.

Une véritable **liaison nulle** (une liaison avec un nom d'utilisateur mais un mot de passe vide) est une mauvaise configuration apparentée que certains annuaires acceptent comme anonyme ; testez-la lorsqu'une liaison simple avec un mot de passe vide est proposée.

## Notes par produit

- **Active Directory** : le rootDSE (contextes de nommage, contrôles pris en charge) est presque toujours lisible anonymement, ce qui suffit pour la [découverte d'hôtes et de domaine](../../active-directory/authentication/credentials/host-and-domain-discovery.md) ; les données d'objets ne sont anonymes que si `dsHeuristics` a été assoupli ou si le groupe de compatibilité pré-Windows-2000 inclut Anonymous.
- **OpenLDAP / 389 DS** : la lecture anonyme de tout l'arbre est un défaut ou une mauvaise configuration fréquents ; les empreintes `userPassword` peuvent même être renvoyées anonymement sur un serveur aux ACL mal réglées.
- **Équipements** (stockage, matériel réseau, applications à LDAP embarqué) sont souvent livrés avec un accès anonyme permissif.

## Notes d'exploitation

- Lisez toujours le rootDSE en premier ; même lorsque les lectures de données sont bloquées, il donne les contextes de nommage, les mécanismes SASL pris en charge et l'identité du serveur.
- Sur OpenLDAP, demandez `userPassword` explicitement ; là où l'ACL est erronée, les empreintes de mots de passe reviennent anonymement et se cassent hors ligne.
- Les lectures anonymes alimentent tout le reste : la liste des utilisateurs pour la pulvérisation, les e-mails pour l'hameçonnage, et la structure de l'annuaire pour le ciblage (voir [identifiants dans les attributs](credentials-in-attributes.md)).

## Outils

- **ldapsearch** : lectures par liaison anonyme et nulle.
- **NetExec (nxc) ldap** : utilitaires d'énumération anonyme.
- **windapsearch / ldapdomaindump** : extractions structurées anonymes/authentifiées là où c'est permis.

## Références

- The Hacker Recipes: LDAP enumeration
- RFC 4513: LDAP authentication methods (anonymous and unauthenticated bind)
