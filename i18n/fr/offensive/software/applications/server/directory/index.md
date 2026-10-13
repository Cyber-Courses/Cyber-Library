---
title: "Services d'annuaire : attaquer Active Directory et les annuaires LDAP"
order: 3
description: "Techniques offensives contre les services d'annuaire : la surface d'attaque Active Directory (énumération, abus d'authentification et d'identifiants, ACL, approbations, persistance) et les attaques génériques contre les annuaires LDAP."
keywords:
  - active directory
  - LDAP
  - directory services
  - domain compromise
  - AD attacks
---

# Annuaire

Un **service d'annuaire** stocke et dessert les identités, les identifiants et les données d'autorisation sur lesquels une organisation fonctionne. Le compromettre tient rarement à une seule faille ; il s'agit d'abuser des fonctionnalités, des protocoles et des permissions d'objets du service lui-même pour passer d'un point d'ancrage peu privilégié au contrôle du magasin d'identités, et avec lui de l'ensemble du parc qui lui fait confiance.

Ce domaine se divise par produit d'annuaire, car la surface d'attaque est propre à chaque produit :

- **[Active Directory](active-directory/index.md)**, l'annuaire Windows et son écosystème d'authentification (Kerberos, NTLM, LDAP, AD CS). C'est la cible dominante dans les réseaux d'entreprise et le plus grand sous-arbre.
- **[LDAP](ldap/index.md)**, le protocole d'annuaire générique tel qu'il apparaît sur les annuaires non-Windows (OpenLDAP, 389 Directory Server, et d'autres) et en tant que couche LDAP qu'AD lui-même expose.

## Portée et coutures

Les attaques d'annuaire se situent à la couche service : vous attaquez l'annuaire *en tant que service en cours d'exécution et magasin de données*, pas le traitement des requêtes d'une application. L'**injection LDAP** de couche applicative (une entrée utilisateur concaténée dans un filtre LDAP au sein de code web) est une autre catégorie et se trouve sous [injection de code web](../web/code/injection/database/ldap/index.md) ; ce domaine la met en référence croisée plutôt que de la répéter.

Plusieurs attaques nommées qui passent *à travers* un domaine ciblent en réalité des produits distincts, elles résident donc dans leurs propres domaines serveur et sont mises en référence croisée depuis Active Directory : le protocole Netlogon, le service Print Spooler, Exchange, et les plateformes de gestion de configuration telles que SCCM.

**Microsoft Entra ID** (anciennement Azure AD), le tenant cloud vers lequel Active Directory se synchronise couramment, est un annuaire hébergé par l'éditeur et est attaqué sous [Online > Identity](../../online/identity/entra-id/index.md). Le côté Active Directory on-premises et le pont de synchronisation Entra Connect restent ici et le mettent en référence croisée.

## Sections

- **[Active Directory](active-directory/index.md)** : énumération, abus d'authentification et d'identifiants, abus de DACL, stratégie de groupe, approbations et persistance.
- **[LDAP](ldap/index.md)** : énumération anonyme et authentifiée, identifiants exposés dans les attributs, et faiblesses de signature et de liaison de canal.

## Références

- The Hacker Recipes: Active Directory
- PortSwigger / OWASP WSTG: Identity and authentication testing
