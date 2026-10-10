---
title: "Annuaires LDAP : attaquer le protocole d'annuaire et ses services"
description: "Techniques offensives contre les services d'annuaire LDAP (AD, OpenLDAP, 389 Directory Server) : énumération anonyme et authentifiée, identifiants exposés dans les attributs, et faiblesses de signature LDAP et de liaison de canal."
keywords:
  - LDAP
  - anonymous bind
  - OpenLDAP
  - LDAP signing
  - channel binding
---

# LDAP

LDAP est le protocole de requête que parlent les annuaires, que l'annuaire soit Active Directory, OpenLDAP, 389 Directory Server, ou l'annuaire embarqué d'un équipement. Cette section couvre les attaques qui ciblent le service LDAP lui-même plutôt qu'un produit spécifique : lire l'annuaire avec ou sans identifiants, moissonner les secrets que les administrateurs stockent dans les attributs d'objets, et les faiblesses de transport (absence de signature et de liaison de canal) qui rendent le service relayable.

Celles-ci s'appliquent à tous les produits. Le modèle d'objets et les chaînes d'attaque propres à AD résident sous [Active Directory](../active-directory/index.md) ; cette section est le foyer du protocole lui-même et des annuaires non-AD.

## Où ça casse

- **Lectures trop permissives.** Les annuaires autorisent fréquemment les liaisons anonymes ou nulles, et même les lectures authentifiées exposent bien plus que prévu, livrant l'inventaire des utilisateurs et des groupes ainsi que la structure de l'annuaire.
- **Secrets dans les attributs.** Les administrateurs cachent des mots de passe dans `description`, `userPassword`, `info` et des champs personnalisés, et AD stocke les mots de passe de machines et de services dans les attributs LAPS et gMSA lisibles par certains principaux.
- **Transport faible.** Un LDAP qui n'exige ni signature ni liaison de canal peut être relayé, transformant une authentification contrainte ou capturée en écritures dans l'annuaire.

## Sections

- **[Protocole](protocol/index.md)** : les attaques indépendantes de l'implémentation qui fonctionnent contre tout annuaire LDAP, y compris l'énumération anonyme et authentifiée, les identifiants stockés dans les attributs, et les faiblesses de signature et de liaison de canal.

La couverture par implémentation pour OpenLDAP, 389 Directory Server et eDirectory est organisée aux côtés de Protocole à mesure que ces pages sont rédigées.

## Références

- The Hacker Recipes: LDAP
- RFC 4511: Lightweight Directory Access Protocol (LDAP)
