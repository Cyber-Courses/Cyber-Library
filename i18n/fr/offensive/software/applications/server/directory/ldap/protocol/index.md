---
title: "Attaques du protocole LDAP"
description: "Attaques LDAP qui ciblent le protocole plutôt que le modèle d'objets d'un produit : binds et énumération anonymes et authentifiés, identifiants stockés dans les attributs des objets, et faiblesses de la signature LDAP et du channel binding (ces dernières spécifiques aux annuaires s'authentifiant par NTLM comme Active Directory)."
keywords:
  - LDAP
  - bind anonyme
  - énumération LDAP
  - signature LDAP
  - channel binding
---

# Attaques du protocole

Ces attaques ciblent le service LDAP lui-même plutôt que le modèle d'objets d'un produit. Lire l'annuaire avec ou sans identifiants, et récolter les secrets que les administrateurs laissent dans les attributs des objets, fonctionnent contre tout annuaire qui parle le protocole, qu'il s'agisse d'Active Directory, OpenLDAP, 389 Directory Server ou de l'annuaire embarqué d'un appliance. Les faiblesses de transport sont plus étroites : la signature LDAP et le channel binding verrouillent un relais NTLM vers l'annuaire, ils s'appliquent donc à Active Directory et aux autres annuaires intégrés à Windows, pas à OpenLDAP ni 389 Directory Server en tant que tels.

Les modèles d'objets et chaînes d'attaque spécifiques aux produits se trouvent ailleurs : [Active Directory](../../active-directory/index.md) pour AD, et les sections par implémentation sous [LDAP](../index.md) pour OpenLDAP, 389 Directory Server et eDirectory.

## Pages

- **[Bind et énumération anonymes](anonymous-bind-and-enumeration.md)** : lire un annuaire avec un bind null ou anonyme.
- **[Énumération authentifiée](authenticated-enumeration.md)** : extraire l'ensemble complet des objets avec n'importe quel bind valide.
- **[Identifiants dans les attributs](credentials-in-attributes.md)** : mots de passe et secrets gérés stockés dans les champs des objets.
- **[Signature LDAP et channel binding](signing-and-channel-binding.md)** : les faiblesses de transport qui permettent le relais.

## Références

- [The Hacker Recipes: LDAP](https://www.thehacker.recipes/ad/recon/ldap)
- [RFC 4511: Lightweight Directory Access Protocol (LDAP): The Protocol](https://www.rfc-editor.org/rfc/rfc4511)
